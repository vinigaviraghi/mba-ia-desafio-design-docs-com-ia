# Tracker

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | docs/PRD.md | Objetivo | Entrega de evento elegível em menos de 10s para endpoint disponível. | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| PRD-OBJ-02 | docs/PRD.md | Objetivo | Worker consulta eventos pendentes em intervalos de 2s. | TRANSCRICAO | [09:09] Diego; [09:10] Larissa |
| PRD-OBJ-03 | docs/PRD.md | Objetivo | Meta de zero perda de eventos em testes ponta a ponta. | TRANSCRICAO | [09:04] Bruno; [09:48] Larissa |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | CRUD de webhook com URL, eventos e estado ativo. | TRANSCRICAO | [09:31] Marcos; [09:33] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Secret gerada na criação do webhook. | TRANSCRICAO | [09:31] Marcos; [09:21] Sofia |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Listar, alterar e excluir webhooks. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Filtrar quais status disparam cada webhook. | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Toda mudança elegível de status gera evento outbound. | TRANSCRICAO | [09:00] Marcos; [09:40] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Registrar histórico de entrega com payload, response e duração. | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Rotação de secret com janela de coexistência. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Secret antiga válida por 24 horas após rotação. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Falhas repetidas vão para DLQ. | TRANSCRICAO | [09:15] Diego; [09:18] Diego |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | ADMIN reprocessa item da DLQ. | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | URLs devem usar HTTPS. | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 segundos no worker. | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Payload limitado a 64 KB. | TRANSCRICAO | [09:23] Sofia; [09:24] Diego |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Semântica at-least-once. | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Correlacionar request, event, delivery e replay. | TRANSCRICAO | [09:19] Sofia; [09:36] Sofia |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Reusar MySQL, Prisma, JWT e logger existentes. | CODIGO | `package.json`; `src/shared/logger/index.ts`; `src/middlewares/auth.middleware.ts` |
| PRD-RISK-01 | docs/PRD.md | Risco | Endpoint indisponível por longos períodos. | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| PRD-RISK-02 | docs/PRD.md | Risco | Crescimento da outbox e da DLQ. | TRANSCRICAO | [09:08] Diego |
| PRD-RISK-03 | docs/PRD.md | Risco | Integração errada com HMAC/secret. | TRANSCRICAO | [09:19] Sofia; [09:21] Sofia |
| PRD-AC-01 | docs/PRD.md | Critério de Aceitação | Mudança de status permanece transacional com outbox. | TRANSCRICAO | [09:40] Bruno; [09:41] Bruno |
| PRD-AC-02 | docs/PRD.md | Critério de Aceitação | CRUD de webhooks via API. | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno |
| PRD-AC-03 | docs/PRD.md | Critério de Aceitação | Entrega com HMAC e headers de rastreio. | TRANSCRICAO | [09:20] Sofia; [09:44] Diego |
| PRD-AC-04 | docs/PRD.md | Critério de Aceitação | Replay de DLQ por ADMIN com log. | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-05 | docs/PRD.md | Critério de Aceitação | Histórico de entrega consultável. | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-06 | docs/PRD.md | Critério de Aceitação | Nenhum requisito inventado contradiz a transcrição. | TRANSCRICAO | [09:48] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa | Chamada síncrona no `changeStatus` descartada. | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa | Redis Streams / broker externo descartado. | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa | Trigger de banco para disparo descartada. | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-Q-01 | docs/RFC.md | Questão em Aberto | Retenção de entregas e DLQ não fechada. | TRANSCRICAO | [09:08] Diego |
| RFC-Q-02 | docs/RFC.md | Questão em Aberto | Scale-out do worker ficou para depois. | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| RFC-Q-03 | docs/RFC.md | Questão em Aberto | Política de rate limiting de saída por cliente. | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL para registrar eventos. | TRANSCRICAO | [09:06] Diego; [09:07] Diego |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker separado em polling a cada 2 segundos. | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Retry com 5 tentativas, backoff e DLQ. | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-por-endpoint.md | Decisão | HMAC-SHA256 com secret por endpoint e rotação. | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia |
| ADR-005 | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Decisão | At-least-once com `X-Event-Id` para dedupe. | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso dos padrões do projeto e módulos existentes. | CODIGO | `src/app.ts`; `src/middlewares/error.middleware.ts`; `src/middlewares/auth.middleware.ts`; `src/shared/logger/index.ts` |
| ADR-007 | docs/adrs/ADR-007-payload-snapshot-na-outbox.md | Decisão | Snapshot do payload na inserção da outbox. | TRANSCRICAO | [09:52] Larissa; [09:52] Diego |
| FDD-FLOW-01 | docs/FDD.md | Fluxo | Evento é gravado na outbox na mesma transação do pedido. | TRANSCRICAO | [09:40] Bruno; [09:41] Bruno |
| FDD-FLOW-02 | docs/FDD.md | Fluxo | Worker faz polling da outbox a cada 2 segundos. | TRANSCRICAO | [09:09] Diego |
| FDD-FLOW-03 | docs/FDD.md | Fluxo | Retry com backoff 1m, 5m, 30m, 2h e 12h. | TRANSCRICAO | [09:17] Diego |
| FDD-FLOW-04 | docs/FDD.md | Fluxo | DLQ e replay manual por ADMIN. | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-API-01 | docs/FDD.md | Contrato | POST `/api/v1/webhooks` cria webhook. | TRANSCRICAO | [09:31] Marcos |
| FDD-API-02 | docs/FDD.md | Contrato | GET `/api/v1/webhooks` lista webhooks. | TRANSCRICAO | [09:33] Bruno |
| FDD-API-03 | docs/FDD.md | Contrato | PATCH `/api/v1/webhooks/:id` atualiza webhook. | TRANSCRICAO | [09:33] Bruno |
| FDD-API-04 | docs/FDD.md | Contrato | POST `/api/v1/webhooks/:id/rotate-secret` rotaciona secret. | TRANSCRICAO | [09:21] Sofia |
| FDD-API-05 | docs/FDD.md | Contrato | GET `/api/v1/webhooks/:id/deliveries` lista entregas. | TRANSCRICAO | [09:34] Marcos |
| FDD-API-06 | docs/FDD.md | Contrato | POST `/api/v1/admin/webhooks/dead-letter/:id/replay` faz replay. | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-ERR-01 | docs/FDD.md | Erro | `WEBHOOK_INVALID_URL` para URL inválida ou sem HTTPS. | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-02 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_TIMEOUT` para timeout de 10s. | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-03 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_RETRY_EXHAUSTED` após 5 tentativas. | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| FDD-ERR-04 | docs/FDD.md | Erro | `WEBHOOK_DLQ_REPLAY_FORBIDDEN` sem role ADMIN. | TRANSCRICAO | [09:36] Sofia |
| FDD-ERR-05 | docs/FDD.md | Erro | `WEBHOOK_PAYLOAD_TOO_LARGE` para payload acima de 64 KB. | TRANSCRICAO | [09:23] Sofia |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Métricas de fila, sucesso, falha e duração. | TRANSCRICAO | [09:08] Diego; [09:48] Larissa |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Logs Pino com requestId/eventId/deliveryId. | CODIGO | `src/shared/logger/index.ts`; `src/middlewares/request-logger.middleware.ts` |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | Tracing por correlação entre request, event e delivery. | TRANSCRICAO | [09:19] Sofia; [09:36] Sofia |
| FDD-INT-01 | docs/FDD.md | Integração | `changeStatus` publica evento na outbox dentro da transação. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-02 | docs/FDD.md | Integração | Máquina de estados define eventos elegíveis. | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-INT-03 | docs/FDD.md | Integração | Auth e `requireRole` são reutilizados. | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-INT-04 | docs/FDD.md | Integração | Error middleware trata erros do módulo. | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-INT-05 | docs/FDD.md | Integração | Router da API recebe novo módulo. | CODIGO | `src/app.ts`; `src/routes/index.ts` |
| FDD-INT-06 | docs/FDD.md | Integração | Prisma schema atual é a base para novas tabelas. | CODIGO | `prisma/schema.prisma` |
| FDD-AC-01 | docs/FDD.md | Critério técnico | Mudança de status continua transacional com outbox. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-AC-02 | docs/FDD.md | Critério técnico | Worker separado não bloqueia API. | TRANSCRICAO | [09:11] Diego |
| FDD-AC-03 | docs/FDD.md | Critério técnico | Headers de entrega incluem event id, signature, timestamp e webhook id. | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| FDD-AC-04 | docs/FDD.md | Critério técnico | Replay exige ADMIN e fica auditável. | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-05 | docs/FDD.md | Critério técnico | Histórico de entrega retorna sucesso, falha, payload e duração. | TRANSCRICAO | [09:34] Marcos |
| FDD-RISK-01 | docs/FDD.md | Risco | Duplicidade eventual de entrega. | TRANSCRICAO | [09:25] Diego |
| FDD-RISK-02 | docs/FDD.md | Risco | Crescimento de outbox e DLQ. | TRANSCRICAO | [09:08] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Integração externa errar HMAC. | TRANSCRICAO | [09:19] Sofia; [09:20] Sofia |
| FDD-RISK-04 | docs/FDD.md | Risco | Worker acumular atraso com volume alto. | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| CODE-01 | docs/FDD.md | Base de código | Padrão de erro centralizado com `AppError` e subclasses. | CODIGO | `src/shared/errors/app-error.ts`; `src/shared/errors/http-errors.ts` |
| CODE-02 | docs/FDD.md | Base de código | Autenticação JWT e `requireRole` já existem. | CODIGO | `src/middlewares/auth.middleware.ts` |
| CODE-03 | docs/FDD.md | Base de código | Logger Pino já redige dados sensíveis. | CODIGO | `src/shared/logger/index.ts` |
| CODE-04 | docs/FDD.md | Base de código | `order.service.ts` já usa transação Prisma para status e estoque. | CODIGO | `src/modules/orders/order.service.ts` |
| CODE-05 | docs/FDD.md | Base de código | API é montada via `buildApiRouter` e `buildApp`. | CODIGO | `src/routes/index.ts`; `src/app.ts`; `src/server.ts` |
