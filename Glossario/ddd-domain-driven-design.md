---
title: "DDD - Domain-Driven Design"
date: 2026-06-18
tags:
  - glossario
  - arquitetura
  - ddd
  - status/rascunho
area: arquitetura
aliases:
  - DDD
  - Domain-Driven Design
  - Design Orientado a Domínio
---

# DDD - Domain-Driven Design

## 📌 Definição

Abordagem de design de software que coloca o **domínio do negócio** no centro das decisões, modelando o código em torno da linguagem e das regras do negócio (linguagem ubíqua, agregados, eventos de domínio, contextos delimitados).

## 🔎 Contexto de uso

Usado para organizar sistemas complexos separando o que é estratégico do que é acessório. Distinção central entre tipos de domínio:

- **Core domain** — onde "você ganha o jogo"; o diferencial competitivo. Concentre aqui o maior esforço e qualidade.
- **Subdomínio de suporte** — necessário para o core funcionar, mas não é diferencial; desenvolve-se só o suficiente.
- **Subdomínio genérico** — todo mundo precisa e funciona igual para todos; em geral **compra-se/terceiriza-se** em vez de construir.

## 💡 Exemplo prático

- **Subdomínio genérico:** pagar imposto. Toda empresa precisa, e funciona igual para todas — por isso o iFood **compra** um sistema fiscal (ex.: Synchro) em vez de construir o próprio: sai mais barato que manter time de devs + especialistas tributários atualizados.
- **No projeto XTPG:** a conexão bancária via Open Finance é tratada como subdomínio genérico → parceria com a [[Pluggy]] em vez de certificar direto no Banco Central. Isso reserva o esforço para o core domain (a tesouraria em si) e permite trocar de agregador isolando `BankConnectivity` de `BankTransaction`.

## Termos relacionados

- [[modelagem-orientada-a-eventos]]
- [[open-finance]]
- [[load-bearing]]
- [[anti-corruption-layer]]

## Fontes

- [[2026-06-18]] — daily XTPG (mentoria Elemar)
