# ADR-001: Outbox no MySQL para registrar eventos de webhook

Status: Aceito
Data: 2026-10-19

## Contexto

A mudança de status de pedido já acontece dentro de uma transação SQL que atualiza `orders`,
registra `order_status_history` e, quando necessário, ajusta estoque. A reunião concluiu que
chamar o endpoint do cliente dentro dessa mesma transação criaria acoplamento e risco de
rollback indevido.

## Decisão

Adotar o padrão outbox no MySQL. Quando um pedido mudar de status, a aplicação vai inserir
um registro de evento em uma tabela de outbox na mesma transação que atualiza o pedido.
Somente depois do commit o worker externo passa a processar esse evento.

O evento será armazenado como snapshot do momento da mudança, com `event_id` em UUID e
status de processamento separado da entidade de pedido.

## Alternativas consideradas

1. Chamada HTTP síncrona dentro de `changeStatus`.
   - Rejeitada porque o endpoint do cliente poderia atrasar ou falhar e bloquear a transação
     principal.
2. Fila externa como Redis Streams.
   - Rejeitada porque exigiria nova infraestrutura sem necessidade real para o escopo atual.

## Consequências

Positivas:
- Mantém atomicidade entre a mudança do pedido e o registro do evento.
- Usa a infraestrutura já existente do MySQL.
- Facilita auditoria e reprocessamento.

Negativas:
- Introduz novas tabelas e um worker de consumo.
- Exige limpeza/retention futura para manter a tabela enxuta.
