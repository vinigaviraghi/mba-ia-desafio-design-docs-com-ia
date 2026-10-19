# Sobre o desafio

Este desafio pediu a transformação de uma reunião técnica em um pacote completo de design docs
para uma feature de webhooks de notificação de pedidos. O trabalho foi começar com a transcrição
e o código existente, separar o que era decisão, requisito, restrição e adiamento, e então
converter isso em documentação técnica rastreável.

O objetivo final não era apenas escrever documentação bonita. Era produzir um conjunto coerente
de artefatos que um time de engenharia pudesse usar para iniciar a implementação sem depender de
suposições externas ou de conteúdo inventado pela IA.

# Ferramentas de IA utilizadas

- `Codex desktop`: principal ferramenta para leitura do repositório, extração de decisões da transcrição e redação dos documentos.
- `Terminal local`: apoio para inspecionar arquivos, confirmar a estrutura do código e validar caminhos reais mencionados na documentação.

# Workflow adotado

1. Li a transcrição inteira e marquei decisões fechadas, pontos adiados e itens explicitamente fora de escopo.
2. Mapeei o código-base para entender os padrões já existentes: módulos, erros, autenticação, logger e a transação de pedidos.
3. Escrevi os ADRs primeiro, porque eles formam a base arquitetural da feature.
4. Consolidei o RFC em cima dessas decisões, mantendo o nível arquitetural sem entrar demais em implementação.
5. Detalhei o FDD com fluxos, contratos, erros, observabilidade e integração com o sistema existente.
6. Fechei o PRD como visão de produto e consolidei o tracker com a origem de cada item.
7. Por fim, substituí este README pelo relato do processo e pela ordem sugerida de leitura.

# Prompts customizados

```text
Leia a transcrição e extraia apenas decisões fechadas, requisitos funcionais, riscos, exclusões e pontos adiados.
Separe claramente o que entra em PRD, RFC, FDD e ADRs. Não invente nada que não esteja na transcrição.
```

```text
Com base na transcrição e nos arquivos reais do repositório, escreva os ADRs da feature de webhooks.
Cada ADR deve conter contexto, decisão, alternativas consideradas e consequências. Referencie os caminhos de código
quando a decisão depender diretamente do padrão existente do projeto.
```

# Iterações e ajustes

O material final passou por 4 ciclos principais:

1. Primeiro, extraí as decisões da reunião e separei o que era produto, arquitetura e implementação.
2. Depois, refinei os ADRs para evitar repetição entre documentos.
3. Em seguida, ajustei o FDD para ficar realmente acionável, com contratos e erros concretos.
4. Por fim, revisei rastreabilidade e consistência para garantir que os itens tivessem origem na transcrição ou no código.

Os principais ajustes foram remover ideias fora de escopo, tornar explícita a integração com o código existente e amarrar cada documento ao tracker.

# Como navegar a entrega

1. [Tracker](./docs/TRACKER.md) para ver a rastreabilidade entre documentos, transcrição e código.
2. [ADRs](./docs/adrs/) para entender as decisões técnicas fechadas.
3. [RFC](./docs/RFC.md) para ler a proposta técnica consolidada.
4. [FDD](./docs/FDD.md) para ver o desenho detalhado de implementação.
5. [PRD](./docs/PRD.md) para fechar a leitura com a visão de produto.
