---
title: 2026-03-28
date: 2026-03-28
tags:
  - guia
  - status/consolidado
  - diario
---

# Guia do Vault — Segundo Cérebro

Este vault é um sistema de gestão de conhecimento pessoal (PKM) projetado para o dia a dia de um engenheiro e aspirante a arquiteto de software. O objetivo é simples: ==capturar, conectar e recuperar conhecimento== de forma que ele se torne útil no momento certo.

> [!tip] Princípio central
> O melhor sistema é aquele que você realmente usa. Mantenha simples, capture rápido, refine depois.

---

## Como o vault está organizado

```
second-brain/
├── Diario/          Registro diário — foco, tarefas, aprendizados
├── Pesquisa/        Estudos aprofundados sobre um tema
├── Referencias/     Fontes externas — artigos, livros, vídeos
├── Glossario/       Termos técnicos com definição e contexto
├── Projetos/        Notas organizadas por projeto ativo
├── Lideranca/       Gestão de time, 1:1s, feedbacks, retrospectivas
├── Arquitetura/     Decisões arquiteturais, ADRs, trade-offs
├── Ideias/          Captura rápida de insights e hipóteses
└── Templates/       Modelos reutilizáveis (não editar diretamente)
```

Cada pasta tem um propósito claro. Quando você souber onde uma informação vive, encontrá-la se torna trivial.

---

## Diário — O hábito mais importante

**Pasta:** `Diario/`
**Formato:** `YYYY-MM-DD.md` (ex: `2026-03-28.md`)
**Template:** `Templates/template-diario.md`

A nota diária é o ponto de entrada do vault. Tudo começa aqui. Abra-a no início do dia e volte a ela ao longo do dia para registrar o que importa.

### O que registrar

| Seção                  | O que colocar                                                              | Quando preencher       |
| ---------------------- | -------------------------------------------------------------------------- | ---------------------- |
| **Foco do dia**        | 1-3 objetivos principais do dia                                            | Início do dia          |
| **Tarefas**            | Checklist do que precisa ser feito                                         | Início do dia          |
| **Reuniões**           | Pontos-chave de reuniões, 1:1s, alinhamentos                              | Durante/após reuniões  |
| **Ideias e insights**  | Qualquer ideia que surgiu — não filtre, apenas capture                     | Ao longo do dia        |
| **O que aprendi**      | Conceitos, técnicas ou informações novas                                   | Ao longo do dia        |
| **Review do dia**      | O que foi bem, o que melhorar, nota para amanhã                            | Final do dia           |

### Dicas práticas

- **Não precisa ser perfeito.** Frases curtas e bullet points são suficientes
- **Capture primeiro, organize depois.** Se uma ideia surge no meio do dia, jogue na nota diária. Depois você move para a pasta certa
- **Use wikilinks.** Se mencionou um conceito que já tem nota, linke: `[[strangler-fig-pattern]]`. Isso cria a rede de conhecimento
- **5 minutos no final do dia** para o review já fazem diferença enorme ao longo de semanas

### Exemplo de fluxo

1. Abra a nota do dia (ou crie a partir do template)
2. Defina o foco e as tarefas
3. Durante o dia, registre reuniões e ideias conforme surgem
4. No final do dia, faça o review em 5 minutos
5. Se alguma ideia merece aprofundamento, crie uma nota em `Pesquisa/` ou `Ideias/`

---

## Pesquisa — Estudos aprofundados

**Pasta:** `Pesquisa/`
**Template:** `Templates/template-pesquisa.md`
**Índice:** [[indice-pesquisa]]

Aqui vivem os estudos que você faz sobre um tema específico. Cada nota de pesquisa responde à pergunta: **"O que eu sei sobre esse assunto e como posso usar esse conhecimento?"**

### Quando criar uma nota de pesquisa

- Você está estudando um pattern, tecnologia ou conceito novo
- Precisa entender um tema para tomar uma decisão arquitetural
- Quer consolidar o que aprendeu de múltiplas fontes sobre um assunto

### Estrutura da nota

| Seção                       | Propósito                                                            |
| --------------------------- | -------------------------------------------------------------------- |
| **Contexto**                | Por que você está estudando isso? Qual problema quer resolver?       |
| **Resumo**                  | Síntese em 3-5 frases — o "elevator pitch" do que aprendeu          |
| **Pontos principais**       | Os conceitos-chave, detalhados com bullets                           |
| **Como aplicar**            | Conexão direta com seu contexto: time, projetos, decisões            |
| **Diagramas**               | Mermaid, imagens, qualquer visualização que ajude                    |
| **Perguntas em aberto**     | O que você ainda não sabe — revisitar depois                         |
| **Fontes**                  | Links para notas em `Referencias/`                                   |
| **Notas relacionadas**      | Wikilinks para pesquisas, glossário ou projetos conectados           |

### Dicas práticas

- **Uma pesquisa = um arquivo.** Não tente cobrir tudo em uma nota só
- **"Como aplicar" é a seção mais valiosa.** Conhecimento sem ação é apenas trivia
- **Use diagramas Mermaid.** Um diagrama vale mais que 3 parágrafos de explicação
- **Linke para o glossário.** Se um termo aparece na pesquisa e está no glossário, conecte-os

> [!example] Exemplos no vault
> - [[ai-gateway]] — estudo sobre o pattern AI Gateway
> - [[estrategias-modernizacao-sistemas]] — comparativo de estratégias de migração

---

## Referências — Fontes externas

**Pasta:** `Referencias/`
**Template:** `Templates/template-referencia.md`
**Índice:** [[indice-referencias]]

Referências são ponteiros para conhecimento externo — artigos, livros, vídeos, repositórios, palestras. A nota de referência **não é uma cópia** do conteúdo original, mas sim o seu resumo pessoal com a conexão ao seu contexto.

### Quando criar uma referência

- Você leu um artigo ou capítulo de livro que vale guardar
- Encontrou um repositório ou ferramenta relevante
- Assistiu uma palestra ou vídeo com insights úteis

### O que registrar

- **Dados da fonte**: tipo, autor, URL, data de acesso
- **Resumo**: 2-3 frases com o essencial
- **Pontos-chave**: os insights mais importantes
- **Como se conecta ao seu contexto**: por que essa fonte importa para você

### Dicas práticas

- **Nomeie com prefixo `referencia-`** para facilitar busca (ex: `referencia-martin-fowler-strangler.md`)
- **Não copie o artigo inteiro.** Resuma com suas palavras — isso força o entendimento
- **Sempre linke a referência na nota de pesquisa** que ela alimenta

---

## Glossário — Termos e definições

**Pasta:** `Glossario/`
**Template:** `Templates/template-glossario.md`
**Índice:** [[indice-glossario]]

O glossário é o dicionário técnico pessoal do vault. Cada termo tem definição, contexto de uso e um exemplo prático. Quando você ou alguém do time perguntar "o que é X?", a resposta está aqui.

### Quando criar um termo

- Você encontrou um conceito novo durante um estudo
- Precisa registrar um termo que aparece com frequência em discussões
- Quer ter uma definição própria e acessível de algo que consultaria no Google repetidamente

### Estrutura da nota

- **Definição**: clara, objetiva, em 1-3 frases
- **Contexto de uso**: onde e quando esse termo aparece
- **Exemplo prático**: um cenário concreto
- **Termos relacionados**: wikilinks para outros termos do glossário
- **Aliases**: variações do nome no front-matter (ex: ACL, Camada Anti-Corrupção)

### Dicas práticas

- **Use aliases no front-matter.** Isso permite que o Obsidian sugira o termo quando você digitar qualquer variação do nome
- **Mantenha definições curtas.** Se precisar de mais profundidade, crie uma nota de pesquisa e linke
- **O glossário é ótimo para onboarding.** Novos membros do time podem consultar termos que surgem em reuniões

> [!example] Exemplos no vault
> - [[strangler-fig-pattern]] — pattern de modernização incremental
> - [[anti-corruption-layer]] — camada de isolamento entre domínios
> - [[rate-limiting]] — controle de taxa de requests
> - [[branch-by-abstraction]] — modernização no nível de código

---

## Projetos — Notas por projeto ativo

**Pasta:** `Projetos/`
**Índice:** [[indice-projetos]]

Cada projeto ativo pode ter uma pasta ou nota dedicada. Aqui você registra decisões, contexto, status e links relevantes de cada projeto que está tocando.

### O que registrar

- Objetivo e escopo do projeto
- Decisões técnicas tomadas e por quê
- Links para PRs, documentos, boards
- Riscos identificados e mitigações
- Dependências com outros times

### Dicas práticas

- **Crie uma nota índice por projeto** com links para as notas detalhadas
- **Linke para pesquisas e glossário** quando uma decisão se basear em um conceito estudado
- **Arquive projetos finalizados** movendo para uma subpasta `Projetos/arquivo/`

---

## Liderança — Gestão do time

**Pasta:** `Lideranca/`

Como tech lead de 6 desenvolvedores, você tem interações frequentes que geram informações valiosas. Essa pasta centraliza tudo relacionado à gestão de pessoas e do time.

### O que registrar

- **1:1s**: pontos discutidos, ações combinadas, evolução de cada pessoa
- **Feedbacks**: feedbacks dados e recebidos, com data e contexto
- **Retrospectivas**: o que o time levantou, ações definidas
- **Planos de desenvolvimento**: objetivos individuais de cada membro do time
- **Decisões de time**: processos, acordos, convenções definidas em conjunto

### Dicas práticas

- **Crie um arquivo por pessoa** para 1:1s (ex: `Lideranca/1-1-joao.md`) e vá adicionando entradas com data
- **Registre feedback no momento.** Feedback sem contexto temporal perde valor
- **Revise as notas de 1:1 antes de cada reunião** — mostra cuidado e continuidade

---

## Arquitetura — Decisões e patterns

**Pasta:** `Arquitetura/`

Aqui vivem as decisões arquiteturais, ADRs (Architecture Decision Records), análises de trade-offs e patterns que você aplica ou estuda.

### O que registrar

- **ADRs**: decisões arquiteturais no formato Status / Contexto / Decisão / Consequências
- **Trade-off analyses**: comparativos entre alternativas técnicas
- **Patterns aplicados**: como e por que um pattern foi usado em um projeto
- **Diagramas de arquitetura**: C4, sequência, componentes

### Dicas práticas

- **ADRs são imutáveis.** Se uma decisão muda, crie um novo ADR que substitui o anterior
- **Linke ADRs para pesquisas e glossário** — mostra a fundamentação da decisão
- **Considere criar um template de ADR** quando começar a usá-los com frequência

---

## Ideias — Captura rápida

**Pasta:** `Ideias/`

O lugar para jogar qualquer ideia, insight ou hipótese que surgir e que não cabe em outra pasta ainda. O único objetivo aqui é ==não perder a ideia==.

### O que registrar

- Ideias para melhorias no sistema, processo ou time
- Hipóteses a validar
- Coisas que você quer explorar mas não tem tempo agora
- Insights de conversas, conferências, podcasts

### Dicas práticas

- **Formato livre.** Pode ser uma frase, um parágrafo, um diagrama
- **Revise semanalmente.** Promova ideias boas para `Pesquisa/` ou `Projetos/`
- **Descarte sem culpa.** Nem toda ideia merece virar algo — e está tudo bem

---

## Templates — Como usar

**Pasta:** `Templates/`

Os templates disponíveis são:

| Template                      | Uso                                        |
| ----------------------------- | ------------------------------------------ |
| `template-diario.md`         | Nota diária                                |
| `template-pesquisa.md`       | Estudo aprofundado sobre um tema           |
| `template-referencia.md`     | Fonte externa (artigo, livro, vídeo, repo) |
| `template-glossario.md`      | Termo técnico com definição e contexto     |
| `template-revisao-semanal.md`| Revisão semanal e higiene do vault         |
| `template-projeto.md`        | Nota índice de projeto ativo               |
| `template-adr.md`            | Registro de decisão arquitetural           |

### Configuração no Obsidian

1. Vá em **Settings > Core plugins > Templates** e ative o plugin
2. Em **Template folder location**, defina `Templates`
3. Para criar uma nota a partir de um template: `Ctrl+P` > "Insert template"

> [!tip] Alternativa: Templater
> O plugin da comunidade **Templater** oferece mais poder (datas automáticas, prompts interativos). Se quiser automação, vale a instalação.

## Notas índice

- [[indice-pesquisa]] — mapa das trilhas de estudo
- [[indice-referencias]] — mapa das fontes externas
- [[indice-glossario]] — mapa dos termos técnicos do vault
- [[indice-projetos]] — mapa dos projetos ativos

## Rotinas

- [[rotina-revisao-semanal-do-vault]] — fluxo semanal para consolidar, limpar e conectar notas

---

## Fluxo de trabalho diário

Aqui está um fluxo prático para incorporar o vault no seu dia a dia:

### Manhã (5 min)

1. Crie a nota diária (`Diario/YYYY-MM-DD.md`)
2. Defina 1-3 focos do dia
3. Liste as tarefas principais

### Durante o dia

4. Registre pontos de reuniões e 1:1s
5. Capture ideias e insights conforme surgem
6. Se encontrar um artigo relevante, crie uma nota rápida em `Referencias/`

### Final do dia (5 min)

7. Preencha "O que aprendi hoje"
8. Faça o review: o que foi bem, o que melhorar
9. Deixe uma nota para amanhã

### Semanal (30 min)

10. Revise as notas diárias da semana
11. Promova ideias relevantes para `Pesquisa/` ou `Projetos/`
12. Atualize termos no `Glossario/` se encontrou conceitos novos
13. Revise notas com tag `#status/rascunho` e atualize para `#status/revisado`

---

## Convenções e boas práticas

### Nomes de arquivo

- Sem acentos, sem espaços, usar hífens
- Bom: `estrategias-modernizacao-sistemas.md`
- Ruim: `Estratégias de Modernização.md`

### Front-matter obrigatório

Todo arquivo deve ter pelo menos:

```yaml
---
date: YYYY-MM-DD
tags:
  - pelo-menos-uma-tag
---
```

### Tags

Use tags hierárquicas para facilitar buscas:

```
#arquitetura/patterns
#arquitetura/modernizacao
#arquitetura/resiliencia
#lideranca/feedback
#lideranca/1-1
#projeto/nome-do-projeto
#status/rascunho
#status/revisado
#status/consolidado
```

### Wikilinks

- Use `[[nota]]` para linkar notas internas — o Obsidian rastreia renomeações automaticamente
- Use `[texto](url)` apenas para links externos
- Linke generosamente: quanto mais conexões, mais útil o vault se torna

### Callouts

Use callouts para destacar informações importantes:

```markdown
> [!tip] Dica
> Texto da dica

> [!warning] Atenção
> Algo a observar

> [!danger] Cuidado
> Risco crítico

> [!example] Exemplo
> Caso prático
```

---

## Por que funciona

O poder de um segundo cérebro não está em nenhuma nota individual — está nas **conexões entre elas**. Quando você linka uma pesquisa sobre [[ai-gateway]] a um termo do glossário como [[rate-limiting]], e depois conecta ambos a uma decisão arquitetural de um projeto, você cria uma rede de conhecimento que:

1. **Reduz retrabalho** — você não estuda a mesma coisa duas vezes
2. **Acelera decisões** — o contexto está a um clique de distância
3. **Documenta raciocínio** — meses depois, você sabe *por que* tomou uma decisão
4. **Escala com você** — quanto mais usa, mais valioso se torna

> [!quote]
> "Your mind is for having ideas, not holding them." — David Allen
