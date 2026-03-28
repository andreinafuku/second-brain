---
title: "Branch by Abstraction"
date: 2026-03-28
tags:
  - glossario
  - arquitetura/patterns
  - arquitetura/modernizacao
area: arquitetura
aliases:
  - BbA
---

# Branch by Abstraction

## Definição

Técnica de modernização onde se introduz uma camada de abstração no código existente, implementa-se a nova versão atrás dessa abstração, e troca-se a implementação quando pronta. Permite migrações no nível de código sem necessidade de roteamento de tráfego externo.

## Contexto de uso

Usado para trocar dependências internas (ORM, biblioteca de mensageria, cliente HTTP) de forma segura e incremental, sem criar branches de longa duração no controle de versão.

## Exemplo prático

Para trocar o ORM de Hibernate para jOOQ: (1) cria-se uma interface `Repository`, (2) implementa-se `HibernateRepository` que encapsula o código atual, (3) implementa-se `JooqRepository`, (4) troca-se a injeção de dependência quando pronto.

## Termos relacionados

- [[strangler-fig-pattern]]
- [[anti-corruption-layer]]

## Fontes

- Paul Hammant — branchbyabstraction.com
- [[estrategias-modernizacao-sistemas]]
