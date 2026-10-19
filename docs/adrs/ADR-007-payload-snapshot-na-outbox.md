# ADR-007: Payload snapshot na inserção da outbox

Status: Aceito
Data: 2026-10-19

## Contexto

Se o payload fosse montado somente no momento do envio, mudanças posteriores no pedido poderiam
fazer o webhook representar um estado diferente daquele que disparou o evento.

## Decisão

Armazenar o payload já renderizado na inserção da outbox, no mesmo instante em que a mudança
de status é persistida.

O payload será enxuto e conterá apenas os campos necessários para o consumidor reagir ao evento,
sem itens do pedido.

## Alternativas consideradas

1. Guardar apenas `order_id` e reconstruir o payload no worker.
   - Rejeitado porque introduz drift temporal entre o evento e o estado lido depois.

## Consequências

Positivas:
- Garante que o webhook reflita o estado do momento do commit.
- Facilita auditoria e replay.

Negativas:
- Aumenta o volume armazenado na outbox.
- Mudanças futuras no formato exigem nova versão de payload, não simples leitura tardia.
