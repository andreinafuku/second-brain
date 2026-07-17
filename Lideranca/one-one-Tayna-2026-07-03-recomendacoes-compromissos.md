---
title: "1:1 com Tayná - Recomendações e Compromissos - 2026-07-03"
date: 2026-07-03
tags:
  - lideranca
  - projeto
  - reuniao
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Tayná
---

# ✅ 1:1 com Tayná — Recomendações e Compromissos (2026-07-03)

> [!info] Contexto
> Compilado das recomendações, decisões e compromissos assumidos no 1:1 entre Andre e Tayná
> (03/07/2026) — os indicativos de próximos passos. Ver
> [[one-one-Tayna-2026-07-03-resumo-aprofundado]].

## 📌 Compromissos com prazo

| Prazo | Compromisso | Responsável |
|---|---|---|
| Segunda-feira (06/07) | **Carlão conversa com a Tayná** para formalizar as novas responsabilidades e (provavelmente) tratar da **revisão da taxa horária** | Carlão |
| A seguir (semana de 06/07) | **Clayton faz a passagem** do Atualiza para a Tayná — **aos poucos**, não tudo de uma vez | Clayton → Tayná |
| Semana que vem | Mota começa os **testes** do Treasury Command | Mota |
| Contínuo | Andre compartilha o **link do Elemar** (sequência de prompts) com a Tayná | Andre |

## 🧭 Decisões / definições

- **Tayná vira dona do portal.** Com o Clayton migrando para os testes do Treasury, a Tayná assume
  o portal no dia a dia; para decisões de arquitetura ou segunda opinião, aciona Andre e Clayton
  ("conversa em três").
- **Tayná vira dona do Atualiza.** Assume o módulo de cotações/índices (.NET, web scraping/API,
  jobs schedulados), com passagem gradual pelo Clayton. Mota já aprovou a ideia, condicionada ao
  aceite dela — que topou.
- **Dois produtos, dois times.** Universe+portal (legado) e Treasury Command (novo), com a regra
  **"quem faz, cuida"**. Júlia e Rodrigo ficam "com o pé nos dois" produtos.
- **Fábio entra no monitoramento.** Quando o time começar a pensar o monitoramento do Treasury, o
  Fábio será incluído (o time gera logs e alinha com ele o que mais oferecer).
- **Modelo de manutenção no início:** é aceitável estar no Q3 (novo produto) fazendo manutenção do
  Q2 — mantendo-se no mesmo produto.

## 🛠️ Recomendações de método (Andre → Tayná)

- **Gravar e transcrever** as conversas de negócio (avisando antes), em vez de anotar — usar
  **Granola**, **MS Teams** ou transcrição via Python. Especialmente com **Daiane, Adolfo e
  Fernanda**, e na **passagem com o Clayton**.
- **Usar a sequência de prompts do Elemar** sobre a transcrição: começar pelo prompt que **corrige
  nomes/termos** antes de resumir; depois gerar resumo, destaques, guia de estudos, frases,
  **glossário** e **tabela ontológica** (para mapear contexto).
- **Gerar glossário** ao conversar com negócio — o Claude complementa termos financeiros
  consultando a internet.
- **Usar e abusar do Claude**, inclusive para código — "ele pega muita coisa" e tem o contexto do
  projeto.

## 🔬 Ações técnicas sugeridas (a cargo da Tayná, como dona)

- **Avaliar subir a versão do Atualiza:** verificar arquitetura e versão atuais, pesquisar (com
  apoio do Claude) se uma versão nova traz funcionalidades/performance; decidir em conjunto (os
  três) antes de executar.
- **Mapear as fontes do Atualiza:** entender como cada fonte funciona (scraping vs. API) — o
  levantamento de novas fontes costuma vir da **Daiane**.
- **Manter o primeiro diagnóstico ponta a ponta** (front-end, back-end, banco) e a disciplina de
  não deixar Alê/Su mexerem no banco sem avisar.

## 💬 Encaminhamento de feedback

- Andre já avisou o Carlão que conversaria com a Tayná primeiro; o Carlão formaliza na segunda.
- A revisão de taxa que a Tayná vinha acompanhando deve ser tratada diretamente pelo Carlão.
- A Tayná deve, ao assumir, chamar Andre e Clayton sempre que precisar trocar ideia — "troca ideia
  junto".

---

Relacionado: [[xtpg]] · [[one-one-Tayna-2026-07-03-analise-critica]]
