# ADR-005: Garantia at-least-once com `X-Event-Id`

Status: Aceito
Data: 2026-08-11

## Contexto

O time decidiu não perseguir exactly-once. A preocupação principal era evitar perda de eventos
sem criar coordenação distribuída complexa entre nossa API e o endpoint do cliente.

## Decisão

Cada evento que entra na outbox recebe um UUID único e é enviado com o header `X-Event-Id`.
A semântica de entrega será at-least-once. Se houver retries ou duplicidade de envio, o cliente
deverá deduplicar usando esse identificador.

## Alternativas consideradas

1. Exactly-once com confirmação bilateral.
   - Rejeitado por exigir coordenação muito mais complexa entre dois sistemas independentes.
2. Idempotência por pedido em vez de por evento.
   - Rejeitada porque um mesmo pedido pode gerar vários eventos legítimos ao longo do tempo.

## Consequências

Positivas:
- Simplifica o desenho do sistema.
- É um padrão conhecido em integrações webhook.

Negativas:
- O cliente precisa implementar deduplicação.
- Alguns consumidores podem observar o mesmo evento mais de uma vez.
