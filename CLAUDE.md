# Second Brain - Regras do Vault

## Regras gerais

- Todo arquivo `.md` DEVE ter front-matter YAML com pelo menos `tags` e `date`
- Nomes de arquivos: sem acentos, sem espaços, usar hífens (ex: `minha-nota.md`). Observação: somente os nomes de arquivos não devem ser acentuados. Os conteúdos (textos), devem sim, seguir a gramática correta, seja da língua portuguesa-brasileira ou inglesa.
- Nomes de pastas: sem acentos, sem espaços, usar hífens quando necessário

## Estrutura de pastas

```
Diario/              → Notas diárias no formato YYYY-MM-DD.md
Pesquisa/            → Notas de estudo e pesquisa aprofundada
Referencias/         → Links, artigos, livros, vídeos com resumo
Glossario/           → Termos técnicos com definição e contexto
Projetos/            → Notas por projeto ativo
Lideranca/           → Notas sobre gestão de time, 1:1s, feedbacks
Arquitetura/         → Patterns, decisões, ADRs, trade-offs
Ideias/              → Captura rápida de ideias e insights
Templates/           → Templates de notas (não editar diretamente)
```

## Estilo de conteúdo

- Emojis em headings são bem-vindos — melhoram a escaneabilidade das notas
- Usar emojis como âncora visual nas seções principais (##) e subseções (###)
- Não usar emojis no front-matter, em nomes de arquivo ou em nomes de pasta

## Convenções de conteúdo

- Usar wikilinks `[[nota]]` para conectar notas internas
- Usar tags hierárquicas: `#area/subarea` (ex: `#arquitetura/patterns`)
- Notas diárias sempre em `Diario/YYYY-MM-DD.md`
- Uma pesquisa = um arquivo em `Pesquisa/`
- Um termo = um arquivo em `Glossario/`
- Callouts para destacar decisões, warnings e insights
- Blocos de código com linguagem especificada

## Tags padrão

- `#diario` — notas diárias
- `#pesquisa` — estudos e deep dives
- `#referencia` — fontes externas
- `#glossario` — termos e definições
- `#projeto` — relacionado a projeto ativo
- `#lideranca` — gestão, time, feedback
- `#arquitetura` — decisões e patterns
- `#ideia` — captura rápida
- `#status/rascunho` — nota incompleta
- `#status/revisado` — nota revisada
- `#status/consolidado` — nota finalizada
