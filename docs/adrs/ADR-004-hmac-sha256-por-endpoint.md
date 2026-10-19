# ADR-004: HMAC-SHA256 com secret por endpoint

Status: Aceito
Data: 2026-10-19

## Contexto

Os clientes precisam validar que o webhook veio realmente da nossa plataforma e que o payload
não foi adulterado em trânsito. A reunião também destacou que uma secret única para toda a
plataforma aumentaria o impacto de vazamento.

## Decisão

Assinar o corpo do webhook com HMAC-SHA256 usando uma secret exclusiva por endpoint.
As requisições serão enviadas apenas para URLs HTTPS.

A secret será rotacionável, com período de convivência de 24 horas entre a chave antiga e a nova.

## Alternativas consideradas

1. Secret global da plataforma.
   - Rejeitada porque ampliaria o blast radius de qualquer vazamento.
2. Assinatura assimétrica com chave pública/privada.
   - Rejeitada por aumentar complexidade sem necessidade para a primeira fase.

## Consequências

Positivas:
- Reduz impacto de vazamento de credenciais.
- Mantém verificação simples para o cliente.

Negativas:
- Exige armazenamento e rotação segura das secrets.
- Precisa de documentação clara para integração externa.
