# ADR-003: Retry com backoff exponencial e DLQ

Status: Aceito
Data: 2026-10-19

## Contexto

Endpoints de clientes podem ficar indisponíveis por horas. A solução precisava evitar perda
de eventos e, ao mesmo tempo, impedir retries infinitos que deixariam registros presos para
sempre.

## Decisão

Adotar 5 tentativas de envio com backoff de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas.
Após a última falha, o evento vai para uma tabela de dead letter queue (DLQ) separada.

O reprocessamento da DLQ será manual, via endpoint administrativo.

## Alternativas consideradas

1. Retry infinito com backoff.
   - Rejeitado porque manteria eventos pendurados indefinidamente e dificultaria suporte.
2. Retry fixo sem crescimento de intervalo.
   - Rejeitado porque aumentaria ruído sobre endpoints indisponíveis.

## Consequências

Positivas:
- Limita custo operacional de falhas persistentes.
- Mantém evidência do problema para suporte e auditoria.

Negativas:
- Um evento pode levar até muitas horas para ser marcado como falha permanente.
- Exige fluxo de replay e monitoramento da DLQ.
