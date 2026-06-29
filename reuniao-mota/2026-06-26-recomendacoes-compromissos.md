---
title: "Conversa com Mota - Recomendacoes e Compromissos - 2026-06-26"
date: 2026-06-26
tags:
  - projeto
  - reuniao
  - lideranca
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Mota
---

# ✅ Conversa com Mota — Recomendações e Compromissos Assumidos (2026-06-26)

> [!info] O que é esta nota
> Compilado dos próximos passos, recomendações e compromissos firmados na reunião — o "to-do"
> da conversa. Cada item indica o **responsável**, o **prazo** (quando houve) e o **status**.

## 🎯 Compromissos com prazo

| # | Compromisso | Responsável | Prazo | Status |
|---|-------------|-------------|-------|--------|
| 1 | **Entregar a primeira fase** do produto (épicos Q0–Q4; faltam cadastros e investimentos + débito técnico pequeno) | Andre + Clayton | ~2 semanas | 🟡 Em andamento |
| 2 | **Dar status no chat** sobre a posição da entrega, sem esperar a sexta-feira | Andre | Entre segunda e quarta | 🔵 A fazer |
| 3 | **Conversar com a Alessandra** sobre o teste de exportação de PDF / conciliação (diferença entre conciliado e extrato bancário) | Andre | "Mais tarde" (mesmo dia) | 🔵 A fazer |
| 4 | **Abrir conta no Bradesco e no Santander** (já há Itaú e Banco do Brasil) para ter os 4 maiores bancos no teste de Open Finance | Vitória | Em providência | 🟡 Em andamento |
| 5 | **Buscar e enviar o artigo do Elemar Junior** sobre transcrição/especificação na era da IA | Andre | Na sequência | 🔵 A fazer |

## 🔧 Recomendações técnicas e de processo (acordadas)

| # | Recomendação | Origem / contexto |
|---|--------------|-------------------|
| 6 | **Regra de alerta do cockpit é global**, nunca por empresa. A **segurança por empresa** é aplicada como **filtro na exibição** (via seletor de entidades), não na definição da regra. *(Registrar como ADR.)* | Decisão do Mota, validada por Andre |
| 7 | Manter as **specs e planos (markdown) como fonte da verdade** para a IA; o **kanban** serve só para o humano enxergar progresso, sem duplicar critérios de aceite | Acordo Andre × Claude, endossado pelo Mota |
| 8 | **Não gravar nada na pasta do protótipo do Mota** dentro do repositório — ela é somente leitura/referência | Regra combinada por Andre |
| 9 | Pedir ao Claude para **rever a pasta do protótipo do Mota**, pois houve refinamentos no cockpit de investimentos desde a última análise | Andre, após o Mota avisar das mudanças |
| 10 | Ao paralelizar, usar **no máximo 2 sessões de Docker** simultâneas (limite indicado pela própria ferramenta ao ler a máquina) | Orientação da IA a Andre |
| 11 | Usar **git worktree** para isolar processos paralelos em pastas/branches separadas e evitar conflito de arquivos | Aprendizado de Andre |
| 12 | Padronizar a captura de especificação como **transcrição + Markdown** ("esquece Excel e Word"); usar a IA para **dar nota** à qualidade da spec e devolver pontos obscuros ao negócio | Receita do Elemar Junior, aplicada por Andre/Adolfo |

## 🚀 Encaminhamentos pós-entrega

| # | Encaminhamento | Responsável | Contexto |
|---|----------------|-------------|----------|
| 13 | Subir o **primeiro cliente no XTPG**, isolado, no ambiente **Oracle/Kubernetes** (produtivo) | Andre / time | Para ver dados reais e exercitar a dinâmica de Open Finance |
| 14 | **Falar com o Fábio** para isolar o ambiente do cliente | Andre | Operacionalizar o ambiente produtivo |
| 15 | Fechar o **escopo isolado** do primeiro cliente | Mota | "Você entra lá e fecha o escopo" |

## 🤝 Compromissos de disponibilidade e logística

- **Mota — disponibilidade total:** dúvidas de negócio são **prioridade máxima**, "a qualquer
  momento, sem precisar marcar — chama no chat e a gente vai, quantas vezes precisar".
- **Andre — pode precisar do Mota na próxima semana** para tirar dúvidas de negócio referentes aos
  **débitos/dívidas técnicas** acumulados.
- **Presença no escritório dispensada:** Andre e Clayton podem ficar em "reclusão total"; só
  aparecem se acharem necessário.
- **Ausências previstas do Carlão:**
  - **Segunda-feira:** adiantou um compromisso para as 10h por causa do jogo do Brasil (14h).
  - **Sexta-feira seguinte:** segunda microcirurgia no dentista — provavelmente fora.

> [!note] Notas relacionadas
> [[reuniao-mota/2026-06-26-resumo-aprofundado]] · [[xtpg]] · [[cockpit]] · [[open-finance]]
