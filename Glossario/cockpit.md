---
title: "Cockpit"
date: 2026-06-18
tags:
  - glossario
  - ux
  - produto
  - status/rascunho
area: produto
aliases:
  - Cockpit
---

# Cockpit

## 📌 Definição

No contexto do produto, **tela consolidada de gestão** que agrega informações transacionais em uma visão de alto nível — KPIs/grandes números, painéis de consolidação e regras de **alertas e oportunidades** — para apoiar a tomada de decisão, sem o detalhe operacional registro a registro.

## 🔎 Contexto de uso

Padrão de tela reaproveitado em vários módulos do novo sistema da XTPG. Os cockpits **não criam dado** — são **consolidadores** que olham para as tabelas transacionais (Investments, Intraday Position) e derivam indicadores. O que muda entre eles são as **regras** monitoradas.

## 💡 Exemplo prático

- **Investments Cockpit:** total investido, liquidez imediata, vencimentos, rentabilidade média, concentração; alerta de concentração por banco/emissor acima de X% ou operação abaixo de 95% do CDI.
- **Intraday Cockpit:** total por banco/empresa, saldo por faixa de liquidez; alerta de **queima de caixa** (saída líquida do dia) e **descoberto** (saldo < 0).

## Termos relacionados

- [[today-at-a-glance]]
- [[cash-pooling]]

## Fontes

- [[2026-06-10]] — conversa com Mota (Investments / Intraday cockpits)
