---
title: "Load-Bearing"
date: 2026-06-18
tags:
  - glossario
  - arquitetura
  - status/rascunho
area: arquitetura
aliases:
  - Load Bearing
  - Load-Bearing
  - Peça estrutural
---

# Load-Bearing

## 📌 Definição

Metáfora vinda da engenharia civil: uma **parede portante** (*load-bearing wall*) sustenta o peso da construção acima dela — removê-la faz o prédio desabar, ao contrário de uma parede divisória, que pode sair sem consequência.

No jargão de software, dizer que um elemento é **load-bearing** significa que ele **sustenta outra coisa importante**: não é decorativo nem incidental. Se faltar ou estiver errado, **tudo que depende dele quebra**. Usar o termo é sinal de "isto é peça estrutural, não acabamento — trate antes de prosseguir".

## 🔎 Contexto de uso

Aparece em discussões de arquitetura e revisão de backlog para sinalizar dependências críticas que precisam ser resolvidas **antes** das demais. É uma forma rápida de comunicar prioridade técnica (não de negócio).

## 🧱 Exemplo prático

No projeto XTPG, a US **INVEST-008** faz o de-para entre o **ID da conta na [[Pluggy]]** e o **ID interno** do nosso sistema. Essa tradução é **load-bearing** para o read model de saldo: como os saldos são somente leitura e dependem de casar cada transação ingerida com a conta correta, **sem essa chave de junção as transações não têm como ser atribuídas e o saldo não fecha**. Por isso a INVEST-008 precisa ser priorizada antes do épico que depende dela.

> [!tip] Analogia
> É aquela parede que você **não pode** quebrar ao reformar o apartamento — se quebrar, o teto cai. A parede entre a cozinha e a sala, em geral, você pode.

## Termos relacionados

- [[ddd-domain-driven-design]]
- [[open-finance]]

## Fontes

- [[2026-06-18]] — daily XTPG, glossário de mentoria (Elemar)
