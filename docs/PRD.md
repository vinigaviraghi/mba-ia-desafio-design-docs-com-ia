# PRD — Product Requirements Document

**Sistema de Webhooks de Notificação de Pedidos**

## Resumo e contexto da feature

O sistema vai permitir que clientes B2B recebam notificações quando o status de um pedido
mudar na plataforma. A dor atual é o polling manual em `GET /orders`, que aumenta custo e
atraso de integração. A reunião definiu que a meta é entregar webhooks outbound, sem fluxo
de envio reverso.

## Problema e motivação

Hoje os três clientes priorizados pela operação ficam consultando pedidos repetidamente
para detectar mudanças. Isso gera:

- latência percebida alta;
- custo extra de integração para o cliente;
- risco comercial, já que um dos clientes sinalizou migração caso a feature não chegue a tempo.

O webhook resolve esse problema ao empurrar o evento para o integrador assim que o pedido muda.

## Público-alvo e cenários de uso

- Equipes técnicas dos clientes B2B que integram ERP, OMS ou dashboards internos.
- Time de suporte e operações do cliente, que diagnostica falhas de entrega.
- Time interno de plataforma, que observa retries, DLQ e replay.

### Cenários principais

- Cliente cadastra um endpoint para receber apenas alguns status.
- Pedido muda de `PAID` para `PROCESSING` e o evento é entregue em até poucos segundos.
- Endpoint do cliente fica indisponível e a plataforma reprocessa automaticamente.
- Um item preso na DLQ é reprocessado manualmente por usuário `ADMIN`.

## Objetivos e métricas de sucesso

- `PRD-OBJ-01` a entrega de um evento elegível deve ocorrer em menos de 10 segundos para endpoints disponíveis.
- `PRD-OBJ-02` o worker deve consultar eventos pendentes em intervalos de 2 segundos.
- `PRD-OBJ-03` taxa de perda de eventos igual a zero em testes ponta a ponta com falha de endpoint.
- `PRD-OBJ-04` reduzir a necessidade de polling manual dos clientes que adotarem a integração.

## Escopo

### Incluso

- CRUD de configuração de webhook por cliente.
- Secret gerada pela plataforma na criação.
- Rotação de secret com janela de coexistência de 24 horas.
- Disparo de evento quando um pedido muda de status.
- Histórico de entregas com sucesso, falha, resposta e tempo de resposta.
- Reprocessamento manual de DLQ por `ADMIN`.

### Fora de escopo

- Fallback por e-mail quando o endpoint falhar.
- Dashboard visual para monitorar webhooks.
- Worker com múltiplas instâncias e garantia de ordering global.
- Rate limiting de saída como política fechada nesta fase.

## Requisitos funcionais

| ID | Requisito |
| --- | --- |
| `PRD-FR-01` | O cliente deve conseguir cadastrar um webhook com URL, seleção de eventos e status ativo/inativo. |
| `PRD-FR-02` | A plataforma deve gerar e devolver a secret no momento da criação do webhook. |
| `PRD-FR-03` | O cliente deve conseguir listar, alterar e excluir webhooks cadastrados. |
| `PRD-FR-04` | A plataforma deve permitir filtrar quais status de pedido disparam cada webhook. |
| `PRD-FR-05` | Toda mudança elegível de status deve gerar um evento outbound. |
| `PRD-FR-06` | O sistema deve registrar histórico de entrega com status, payload, response e duração. |
| `PRD-FR-07` | O cliente deve conseguir rotacionar a secret do webhook sem interromper a integração. |
| `PRD-FR-08` | A secret antiga deve permanecer válida por 24 horas após a rotação. |
| `PRD-FR-09` | Eventos que excederem a janela de tentativas devem ir para DLQ. |
| `PRD-FR-10` | Usuário `ADMIN` deve conseguir reprocessar manualmente um item da DLQ. |

## Requisitos não funcionais

| ID | Requisito |
| --- | --- |
| `PRD-NFR-01` | A entrega deve usar HTTPS obrigatório para URLs configuradas. |
| `PRD-NFR-02` | O worker deve considerar timeout de 10 segundos como falha de entrega. |
| `PRD-NFR-03` | O payload enviado ao cliente deve permanecer enxuto e limitado a 64 KB. |
| `PRD-NFR-04` | A semântica de entrega deve ser at-least-once. |
| `PRD-NFR-05` | A observabilidade deve permitir correlacionar request, event, delivery e replay. |
| `PRD-NFR-06` | A solução deve reutilizar MySQL, Prisma, JWT e logger já existentes. |

## Decisões e trade-offs principais

- Outbox no MySQL em vez de HTTP síncrono, para manter atomicidade e evitar travar a transação
  principal.
- Worker separado em polling de 2 segundos, aceitando latência mínima em troca de isolamento.
- HMAC-SHA256 com secret por endpoint, para reduzir blast radius em caso de vazamento.
- At-least-once com `X-Event-Id`, aceitando duplicidade eventual como preço de simplicidade.

## Dependências

- Banco MySQL já usado pela aplicação.
- Prisma Client e migração de schema.
- Middleware de autenticação JWT já existente.
- `ADMIN` já suportado pelo modelo de usuários.
- Logger Pino e middleware central de erros.

## Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- | --- |
| `PRD-RISK-01` | Endpoint do cliente ficar fora do ar por longos períodos. | Média | Alto | Retry com backoff, DLQ e replay manual. |
| `PRD-RISK-02` | Crescimento da outbox e da DLQ sem limpeza. | Média | Médio | Definir retenção e limpeza pós-entrega em fase operacional. |
| `PRD-RISK-03` | Cliente integrar errado a verificação de assinatura. | Média | Alto | Documentação clara, exemplo de HMAC e rotação de secret. |

## Critérios de aceitação

| ID | Critério |
| --- | --- |
| `PRD-AC-01` | Um pedido elegível gera evento de webhook sem perder a atomicidade da mudança de status. |
| `PRD-AC-02` | O cliente consegue cadastrar, editar, listar e excluir um webhook via API. |
| `PRD-AC-03` | A entrega ocorre com HMAC-SHA256 e headers de rastreio definidos. |
| `PRD-AC-04` | Eventos com falha repetida vão para DLQ e podem ser reprocessados por `ADMIN`. |
| `PRD-AC-05` | O histórico de entregas pode ser consultado com sucesso e falha. |
| `PRD-AC-06` | A documentação não registra nenhum requisito fora do que foi discutido na transcrição. |

## Estratégia de testes e validação

- Testes unitários para seleção de eventos, assinatura e transformação de payload.
- Testes de integração para criação da outbox dentro da transação do pedido.
- Testes de contrato para os endpoints de configuração e replay.
- Testes de resiliência para timeout, falha de rede e reprocessamento.
- Revisão de segurança para secret, HMAC, rotação e uso obrigatório de HTTPS.
