# ADR-006: Reuso dos padrões existentes do projeto

Status: Aceito
Data: 2026-10-19

## Contexto

O repositório já possui padrões consolidados para módulos, erros, autenticação, validação,
logger e rotas. A reunião pediu explicitamente que o módulo de webhooks se encaixasse nesse
desenho para reduzir custo de manutenção.

## Decisão

O módulo de webhooks seguirá o mesmo estilo do restante da aplicação:

- módulos em `src/modules/*`
- controllers, services, repositories, routes e schemas com Zod
- erros derivados de `AppError`
- logger Pino já existente
- autenticação com `authenticate` e `requireRole`
- integração transacional via Prisma

Referências diretas de integração incluem `src/modules/orders/order.service.ts`,
`src/middlewares/auth.middleware.ts`, `src/middlewares/error.middleware.ts`,
`src/shared/logger/index.ts` e `src/app.ts`.

## Alternativas consideradas

1. Criar abstrações novas e independentes para webhooks.
   - Rejeitada porque aumentaria complexidade e fugiria do padrão do código.

## Consequências

Positivas:
- Reduz curva de aprendizado do time.
- Mantém consistência estrutural e comportamental.

Negativas:
- O módulo herda algumas limitações do padrão atual.
- Há menos liberdade para otimizar a implementação isoladamente.
