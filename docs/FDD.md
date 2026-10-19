# FDD — Feature Design Document

**Implementação do Sistema de Webhooks de Notificação de Pedidos**

## Contexto e motivação técnica

O módulo de webhooks precisa nascer dentro de uma base já organizada em módulos, services,
repositories, rotas e schemas Zod. A motivação técnica é adicionar entrega assíncrona sem
quebrar a transação crítica que hoje já atualiza pedido, histórico e estoque.

## Objetivos técnicos

- Preservar a atomicidade da mudança de status e do registro do evento.
- Garantir entrega com retry, DLQ e replay manual.
- Reaproveitar autenticação, erros, logger e estilo de módulos já existentes.
- Expor uma API previsível para configuração e auditoria de webhooks.

## Escopo e exclusões

### Escopo

- CRUD de endpoints de webhook.
- Rotação de secret.
- Outbox, worker, retry e DLQ.
- Histórico de entregas.
- Replay de DLQ por `ADMIN`.

### Exclusões

- Dashboard visual.
- E-mail como fallback.
- Escala horizontal com múltiplos workers em paralelo.

## Fluxos detalhados

### FDD-FLOW-01: Criação do evento na outbox

Quando `changeStatus` for executado com um novo status elegível:

1. O serviço abre transação Prisma.
2. Atualiza o pedido e grava `order_status_history`.
3. Gera um snapshot do payload do webhook no mesmo instante.
4. Persiste um registro em `webhook_outbox` com `event_id` UUID.
5. Faz commit.

Se a inserção da outbox falhar, toda a transação deve voltar.

### FDD-FLOW-02: Processamento pelo worker

O worker em um entrypoint próprio consulta a outbox a cada 2 segundos e busca um lote pequeno de
eventos pendentes. Para cada item:

1. Marca o registro como em processamento.
2. Carrega o endpoint configurado e sua secret.
3. Monta o payload e os headers.
4. Dispara o POST HTTP para o cliente.
5. Marca como entregue ou agenda retry.

### FDD-FLOW-03: Retry com backoff

Se a chamada falhar por timeout, erro de rede ou resposta não 2xx:

1. Incrementa `attempt_count`.
2. Agenda `next_attempt_at` com a sequência 1m, 5m, 30m, 2h e 12h.
3. Registra o motivo da falha no histórico de entrega.
4. Depois da quinta tentativa, move para DLQ.

### FDD-FLOW-04: DLQ e replay

Eventos permanentes ficam em `webhook_dead_letters`. O replay:

1. Exige autenticação e role `ADMIN`.
2. Registra quem executou a ação.
3. Reinsere o evento na outbox com novo estado de pendência.

## Contratos públicos

### POST `/api/v1/webhooks`

Cria um webhook para um cliente autenticado.

```http
POST /api/v1/webhooks
Authorization: Bearer <token>
Content-Type: application/json

{
  "customerId": "c4f8e7d1-1e53-4cbb-bbc1-7b2df2a2f100",
  "url": "https://client.example.com/webhooks/orders",
  "events": ["order.status_changed"],
  "active": true
}
```

Resposta `201 Created`:

```json
{
  "id": "7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb",
  "customerId": "c4f8e7d1-1e53-4cbb-bbc1-7b2df2a2f100",
  "url": "https://client.example.com/webhooks/orders",
  "events": ["order.status_changed"],
  "active": true,
  "secret": "whsec_8a0f6c0c5c5d4f1f",
  "createdAt": "2026-08-11T12:00:00.000Z"
}
```

### GET `/api/v1/webhooks`

Lista os webhooks do cliente autenticado.

Resposta `200 OK`:

```json
{
  "items": [
    {
      "id": "7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb",
      "url": "https://client.example.com/webhooks/orders",
      "events": ["order.status_changed"],
      "active": true
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 1
}
```

### PATCH `/api/v1/webhooks/:id`

Atualiza URL, eventos ou status ativo/inativo.

```http
PATCH /api/v1/webhooks/7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb
Authorization: Bearer <token>
Content-Type: application/json

{
  "url": "https://client.example.com/webhooks/orders-v2",
  "events": ["order.status_changed", "order.cancelled"],
  "active": true
}
```

Resposta `200 OK`:

```json
{
  "id": "7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb",
  "url": "https://client.example.com/webhooks/orders-v2",
  "events": ["order.status_changed", "order.cancelled"],
  "active": true
}
```

### DELETE `/api/v1/webhooks/:id`

Remove a configuração do webhook.

Resposta `204 No Content`.

### POST `/api/v1/webhooks/:id/rotate-secret`

Gera uma nova secret com período de coexistência de 24 horas.

```http
POST /api/v1/webhooks/7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb/rotate-secret
Authorization: Bearer <token>
Content-Type: application/json

{}
```

Resposta `200 OK`:

```json
{
  "id": "7e8f9bcb-9d4c-4df8-9f0b-2c9c9eb541cb",
  "newSecret": "whsec_3f2a1d5f7b4e4f99",
  "oldSecretValidUntil": "2026-08-12T12:00:00.000Z"
}
```

### GET `/api/v1/webhooks/:id/deliveries`

Retorna os últimos eventos entregues, com payload, response e duração.

Resposta `200 OK`:

```json
{
  "items": [
    {
      "deliveryId": "del_01HZZ",
      "eventId": "fce93a0f-57f2-4de7-8d55-fc4c1c1d7f22",
      "status": "DELIVERED",
      "httpStatus": 200,
      "responseTimeMs": 184,
      "attemptCount": 1
    }
  ]
}
```

### POST `/api/v1/admin/webhooks/dead-letter/:id/replay`

Reinsere um item da DLQ na fila de processamento.

```http
POST /api/v1/admin/webhooks/dead-letter/8f7a8d3d-0c3d-4f44-a1b9-d564d7e5b0dc/replay
Authorization: Bearer <admin-token>
```

Resposta `200 OK`:

```json
{
  "id": "8f7a8d3d-0c3d-4f44-a1b9-d564d7e5b0dc",
  "status": "REQUEUED",
  "replayedBy": "user-123",
  "replayedAt": "2026-08-11T12:30:00.000Z"
}
```

## Contrato de entrega ao endpoint do cliente

### Request outbound

Headers:

- `Content-Type: application/json`
- `X-Event-Id: <uuid>`
- `X-Webhook-Id: <uuid>`
- `X-Timestamp: <iso-8601>`
- `X-Signature: <hmac-sha256>`

Body:

```json
{
  "event_id": "fce93a0f-57f2-4de7-8d55-fc4c1c1d7f22",
  "event_type": "order.status_changed",
  "timestamp": "2026-08-11T12:00:00.000Z",
  "order_id": "b3c8e637-7c78-4cc3-bf0d-96d3d0d3dc6f",
  "order_number": "ORD-000123",
  "from_status": "PAID",
  "to_status": "PROCESSING",
  "customer_id": "c4f8e7d1-1e53-4cbb-bbc1-7b2df2a2f100",
  "total_cents": 154900
}
```

### Semântica de resposta do cliente

- `2xx`: sucesso.
- `non-2xx`: falha, incrementa retry.
- timeout ou erro de rede: falha, incrementa retry.

## Matriz de erros previstos com códigos `WEBHOOK_*`

| Código | HTTP | Quando acontece | Observação |
| --- | --- | --- | --- |
| `WEBHOOK_INVALID_URL` | 400 | URL inválida ou sem HTTPS. | Validação no schema. |
| `WEBHOOK_INVALID_EVENT_FILTER` | 400 | Lista de eventos fora do catálogo. | Bloqueia persistência. |
| `WEBHOOK_ENDPOINT_NOT_FOUND` | 404 | Webhook não existe para o cliente. | Reutiliza padrão de not found. |
| `WEBHOOK_SECRET_ROTATION_CONFLICT` | 409 | Rotação solicitada antes de concluir janela anterior. | Protege contra regressão de chave. |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | 413 | Snapshot ultrapassa 64 KB. | Evento não é enviado. |
| `WEBHOOK_DELIVERY_TIMEOUT` | 504 | Endpoint não responde em 10 s. | Vai para retry. |
| `WEBHOOK_DELIVERY_RETRY_EXHAUSTED` | 422 | Quinta falha concluída. | Move para DLQ. |
| `WEBHOOK_DLQ_NOT_FOUND` | 404 | Replay tenta usar item inexistente. | Endpoint administrativo. |
| `WEBHOOK_DLQ_REPLAY_FORBIDDEN` | 403 | Usuário sem role `ADMIN`. | Reaproveita `requireRole`. |
| `WEBHOOK_SIGNATURE_GENERATION_FAILED` | 500 | Falha interna ao assinar payload. | Erro operacional raro. |

## Estratégias de resiliência

- Timeout de 10 segundos por entrega outbound.
- Retry com backoff exponencial definido em ADR.
- DLQ como destino final para falhas persistentes.
- Snapshot do payload para evitar drift entre status e entrega.
- `X-Event-Id` para deduplicação no consumidor.
- HTTPS obrigatório e payload enxuto para reduzir exposição e custo.

## Observabilidade

### Métricas

- `webhook_outbox_pending_total`
- `webhook_outbox_processing_total`
- `webhook_delivery_success_total`
- `webhook_delivery_failure_total`
- `webhook_delivery_retry_total`
- `webhook_dead_letter_total`
- `webhook_delivery_duration_ms`

### Logs

Usar o logger Pino já existente com campos:

- `requestId`
- `eventId`
- `webhookId`
- `deliveryId`
- `customerId`
- `attemptCount`
- `httpStatus`

### Tracing

O desenho deve preservar correlação entre API request, evento de outbox e tentativa de entrega.
Se a aplicação adotar tracing distribuído no futuro, os spans devem carregar `requestId`,
`eventId` e `deliveryId` como atributos.

## Dependências e compatibilidade

- Compatível com MySQL e Prisma já presentes no projeto.
- Não altera a semântica da API de pedidos além de publicar eventos adicionais.
- Reaproveita JWT, `ADMIN`, logger Pino e middleware central de erros.
- Adiciona novos modelos no schema Prisma para webhooks, outbox, delivery history e DLQ.

## Critérios de aceite técnicos

- `FDD-AC-01` A mudança de status do pedido continua transacional mesmo com a outbox.
- `FDD-AC-02` O worker consegue processar pendências sem bloquear a API.
- `FDD-AC-03` O cliente recebe `X-Event-Id`, `X-Signature`, `X-Timestamp` e `X-Webhook-Id`.
- `FDD-AC-04` O replay de DLQ exige `ADMIN` e fica registrado em log.
- `FDD-AC-05` O histórico de entrega mostra sucesso, falha, payload e duração.
- `FDD-AC-06` O código de erro do módulo sempre usa prefixo `WEBHOOK_`.

## Riscos e mitigação

| ID | Risco | Mitigação |
| --- | --- | --- |
| `FDD-RISK-01` | Eventual duplicidade de entrega. | Documentar at-least-once e deduplicação por `X-Event-Id`. |
| `FDD-RISK-02` | Crescimento de tabelas da outbox e DLQ. | Planejar retenção e limpeza pós-entrega. |
| `FDD-RISK-03` | Integração externa com HMAC incorreto. | Exemplos de código e documentação de headers. |
| `FDD-RISK-04` | Worker acumular atraso com muitos eventos. | Batch pequeno, polling frequente e monitoramento de fila. |

## Integração com o sistema existente

| Caminho | Integração proposta |
| --- | --- |
| `src/modules/orders/order.service.ts` | Extender `changeStatus` para publicar o evento na outbox dentro da transação Prisma. |
| `src/modules/orders/order.status.ts` | Reaproveitar a máquina de estados para decidir quais transições geram webhook. |
| `src/middlewares/auth.middleware.ts` | Reutilizar `authenticate` e `requireRole` nas rotas do módulo e no replay de DLQ. |
| `src/middlewares/error.middleware.ts` | Deixar que `AppError` e `ZodError` continuem sendo serializados no padrão atual. |
| `src/shared/errors/app-error.ts` | Criar erros do módulo com o mesmo contrato de status code, errorCode e details. |
| `src/shared/logger/index.ts` | Registrar entregas, retries e replays com o mesmo logger Pino e redaction existente. |
| `src/app.ts` | Adicionar o roteamento do módulo de webhooks no bootstrap da API. |
| `src/server.ts` | Manter o entrypoint da API, enquanto o worker ganha entrypoint próprio. |
