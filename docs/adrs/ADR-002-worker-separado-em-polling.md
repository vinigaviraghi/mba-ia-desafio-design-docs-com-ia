# ADR-002: Worker separado em polling

Status: Aceito
Data: 2026-08-11

## Contexto

A solução precisa disparar webhooks sem acoplar a entrega ao ciclo de vida da API HTTP.
A reunião definiu que o sistema deve tolerar reinícios da API e continuar processando eventos
pendentes sem intervenção manual.

## Decisão

Executar o processamento de webhooks em um processo Node.js separado, com loop de polling
a cada 2 segundos sobre a outbox.

O worker deve compartilhar a mesma base de dados e o mesmo conjunto de regras do módulo,
mas rodar fora do processo da API principal.

## Alternativas consideradas

1. Worker embutido no processo da API.
   - Rejeitado porque reinícios da API interromperiam o processamento.
2. Trigger de banco para acionar o envio.
   - Rejeitado porque trigger não substitui um processo externo e não resolve o disparo HTTP.

## Consequências

Positivas:
- Separa o ciclo de vida da API do ciclo de vida do processamento.
- Simplifica implantação e observabilidade operacional.

Negativas:
- Adiciona um processo extra para gerenciar.
- Impõe latência mínima de polling de até 2 segundos.
