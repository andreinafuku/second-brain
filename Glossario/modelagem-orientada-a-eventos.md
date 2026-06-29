---
title: "Modelagem Orientada a Eventos"
date: 2026-06-18
tags:
  - glossario
  - arquitetura
  - ddd
  - status/rascunho
area: arquitetura
aliases:
  - Modelagem Orientada a Eventos
  - Event Modeling
  - Eventos de Domínio
  - Domain Events
---

# Modelagem Orientada a Eventos

## 📌 Definição

Forma de modelar o domínio em torno de **eventos de domínio** — fatos relevantes que **aconteceram no passado** e ficam registrados. Os nomes dos eventos são, por convenção, escritos no passado (ex.: "Pagamento Efetuado", "Publicação de API Realizada").

## 🔎 Contexto de uso

Comum em sistemas desenhados com [[ddd-domain-driven-design]], onde a evolução do estado do negócio é descrita pela sequência de eventos que ocorreram, em vez de apenas pelo estado atual. Apoia padrões como read models (projeções somente leitura construídas a partir dos eventos/transações).

## 💡 Exemplo prático

No projeto XTPG, os documentos de módulo gerados pelo PO (Mota) já entram com conceitos de **agregados, domínios e eventos de domínio**, descrevendo o que aconteceu no sistema: "pagamento efetuado", "publicação de API" etc. Os saldos, por exemplo, são tratados como **somente leitura** — um read model que se apoia em casar cada transação ingerida com a conta correta (ver [[load-bearing]]).

## Termos relacionados

- [[ddd-domain-driven-design]]
- [[load-bearing]]

## Fontes

- [[2026-06-18]] — daily XTPG
