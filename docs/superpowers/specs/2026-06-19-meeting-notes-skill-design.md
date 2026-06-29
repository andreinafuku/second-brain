# Design — Skill `meeting-notes`

- **Data:** 2026-06-19
- **Autor:** André (com Claude)
- **Status:** Aprovado para implementação

## Objetivo

Transformar o fluxo manual de 13 prompts (hoje em
`Dailies/como-gerar-resumos-abrangentes-de-reunioes.md`) em uma **skill global e
portável** que orquestra a geração de resumos abrangentes a partir de uma
transcrição de reunião ou daily.

A skill cobre o pipeline completo (todos os 13 prompts), conduz o fluxo com um
checkpoint humano-no-loop no Prompt 1 e um menu de seleção a cada passo, e grava
cada resultado como arquivo na pasta da reunião, adaptando o front-matter às
convenções do projeto atual.

## Decisões de design

| Eixo | Decisão |
|------|---------|
| Escopo | Pipeline completo — todos os 13 prompts |
| Condução | Checkpoint obrigatório no Prompt 1 + menu de seleção a cada passo |
| Saída | Sempre grava uma nota por prompt |
| Destino | Tudo na pasta da reunião (pasta-pai da transcrição) |
| Entrada | Caminho do arquivo de transcrição; data/pasta/nomes derivados dele |
| Arquitetura | **Prompts embarcados** na skill (auto-contida, portável) |
| Localização | Global: `~/.claude/skills/meeting-notes/` |
| Convenções | Detecta e segue `CLAUDE.md`/`AGENTS.md` do projeto; sem isso, padrão genérico |

## Fluxo de execução

```
Invocação: aponta a transcrição
  ex.: reuniao-mota/transcricoes/tr-2026-06-10.md

1. Detecta convenções — lê CLAUDE.md/AGENTS.md do projeto (se houver) para
   definir front-matter, tags, estilo. Sem isso, usa front-matter genérico
   mínimo (title / date / tags).
2. Deriva contexto — data extraída do nome do arquivo, pasta-pai como destino,
   prefixo de nomes = AAAA-MM-DD-<sufixo>.md.
3. Lê a transcrição inteira.
4. [CHECKPOINT] Roda Prompt 1 → lista termos/nomes suspeitos → PARA.
   Usuário responde com correções (Prompt 2). Skill confirma "OK".
5. [MENU] Mostra os 13 prompts numerados. Usuário escolhe o próximo (um por vez).
6. Roda o prompt escolhido → grava AAAA-MM-DD-<sufixo>.md na pasta da reunião,
   aplicando as correções do passo 4 e as convenções do passo 1.
7. Volta ao MENU (passo 5). Repete até o usuário encerrar.
```

Regras:
- Prompt 1 sempre roda primeiro, com parada obrigatória (respeita o humano-no-loop).
- As correções do passo 4 permanecem em contexto e são aplicadas a todos os
  prompts seguintes.
- Um prompt por vez; a skill não emenda prompts automaticamente.

## Mapa de saída (prompt → sufixo de arquivo)

| # | Prompt | Sufixo |
|---|--------|--------|
| 1 | Contexto e correções | *(checkpoint — não gera arquivo)* |
| 2 | Aplicações e correções | *(alimenta os demais — não gera arquivo)* |
| 3 | Resumo abrangente | `-resumo-aprofundado.md` |
| 4 | Destaque da reunião | `-analise-critica.md` |
| 5 | Insights e takeaways | `-insights-takeaways.md` |
| 6 | Guia de estudos | `-guia-estudos.md` |
| 7 | Frases impactantes + contexto | `-frases-impactantes.md` |
| 8 | Recomendações e compromissos | `-recomendacoes-compromissos.md` |
| 9 | Frases instagramáveis | `-frases-instagramaveis.md` |
| 10 | Glossário | `-glossario.md` |
| 11 | Tabela ontológica | `-tabela-ontologica.md` |
| 12 | Perguntas e respostas | `-perguntas-respostas.md` |
| 13 | Abstract (capa) | `-abstract.md` |

Os sufixos 3/4/5 batem com os nomes já usados em `reuniao-mota/`.

## Fora de escopo (YAGNI)

- Sem roteamento canônico (Glossário não vai para `Glossario/`, Guia não vai
  para `Pesquisa/`).
- Sem execução em batch — sempre um prompt por vez via menu.
- Sem salvar/criar a transcrição — a skill recebe um arquivo já existente.
- Sem atualizar índices (`indice-glossario.md` etc.).
