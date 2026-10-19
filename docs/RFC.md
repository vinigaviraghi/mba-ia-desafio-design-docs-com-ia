# RFC — Request for Comments

**Proposta Técnica do Sistema de Webhooks de Notificação de Pedidos**

| Campo | Valor |
| --- | --- |
| Autor | Codex |
| Status | Em revisão |
| Data | 2026-10-19 |
| Revisores | Larissa, Marcos, Bruno, Diego, Sofia |

## TL;DR

Proponho implementar webhooks outbound usando outbox no MySQL, worker separado em polling,
retry com backoff e DLQ, assinatura HMAC-SHA256 por endpoint e semântica at-least-once com
`X-Event-Id`. A proposta preserva a atomicidade da mudança de status do pedido e evita novas
dependências de infraestrutura.

## Contexto e problema

A plataforma já mantém a mudança de status do pedido em transação com histórico e estoque.
Sem um mecanismo assíncrono, qualquer tentativa de entregar webhooks dentro dessa transação
pode degradar a operação principal. O problema de produto é claro: clientes B2B querem
notificação quase em tempo real sem ficar consultando `GET /orders`.

## Proposta técnica

### Visão geral

1. A API continua sendo responsável por mudar o status do pedido.
2. Dentro da mesma transação, o sistema grava um evento na outbox com snapshot do payload.
3. Um worker separado lê eventos pendentes em polling a cada 2 segundos.
4. O worker envia o webhook para o endpoint do cliente com headers assinados.
5. Falhas transitórias são retentadas com backoff; falhas finais vão para DLQ.
6. O reprocessamento da DLQ fica disponível para `ADMIN`.

### Características centrais

- Outbox persistida no MySQL existente.
- Processamento desacoplado da API HTTP.
- `X-Event-Id` para deduplicação do lado do cliente.
- HMAC-SHA256 com secret por endpoint.
- Logs estruturados com `requestId`, `eventId` e `deliveryId`.

## Alternativas consideradas

| ID | Alternativa | Por que foi descartada |
| --- | --- | --- |
| `RFC-ALT-01` | Chamada HTTP síncrona dentro de `changeStatus`. | Aumentaria o tempo e a fragilidade da transação principal. |
| `RFC-ALT-02` | Redis Streams ou broker externo para filas. | Exigiria nova infraestrutura sem ganho proporcional para a fase atual. |
| `RFC-ALT-03` | Trigger do banco para disparar a entrega. | Não resolve o envio HTTP e ainda adiciona complexidade operacional. |

## Questões em aberto

| ID | Questão | Motivo de abertura |
| --- | --- | --- |
| `RFC-Q-01` | Política exata de retenção de registros entregues e DLQ. | A reunião citou limpeza futura, mas não fechou prazo. |
| `RFC-Q-02` | Estratégia futura de scale-out do worker. | Hoje a premissa é single-worker; particionamento ficou para depois. |
| `RFC-Q-03` | Qual será a política de rate limiting de saída por cliente. | A reunião decidiu observar o comportamento e decidir posteriormente. |

## Impacto e riscos

- Impacto positivo: menos polling dos clientes, menor acoplamento e melhor rastreabilidade.
- Impacto operacional: mais tabelas, mais um processo e necessidade de monitorar DLQ.
- Risco principal: cliente ficar offline por longos períodos e acumular eventos retentados.
- Risco secundário: documentação de integração precisa ser objetiva para evitar erro de assinatura.

## Decisões relacionadas

- [ADR-001: Outbox no MySQL](./adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002: Worker separado em polling](./adrs/ADR-002-worker-separado-em-polling.md)
- [ADR-003: Retry com backoff e DLQ](./adrs/ADR-003-retry-com-backoff-e-dlq.md)
- [ADR-004: HMAC-SHA256 por endpoint](./adrs/ADR-004-hmac-sha256-por-endpoint.md)
- [ADR-005: At-least-once com `X-Event-Id`](./adrs/ADR-005-at-least-once-com-x-event-id.md)
- [ADR-006: Reuso dos padrões do projeto](./adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
- [ADR-007: Payload snapshot na outbox](./adrs/ADR-007-payload-snapshot-na-outbox.md)
