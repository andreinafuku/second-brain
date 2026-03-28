---
title: "Anti-Corruption Layer"
date: 2026-03-28
tags:
  - glossario
  - arquitetura/patterns
  - ddd
area: arquitetura
aliases:
  - ACL
  - Camada Anti-Corrupção
---

# Anti-Corruption Layer (ACL)

## Definição

Camada intermediária que traduz e isola o modelo de domínio de um sistema novo em relação a um sistema legado ou externo. Previne que conceitos e estruturas do sistema antigo "contaminem" o novo modelo.

## Contexto de uso

Originário do Domain-Driven Design (DDD), usado em contextos de integração entre bounded contexts ou durante migrações de sistemas legados. Essencial quando dois sistemas possuem modelos de domínio incompatíveis.

## Exemplo prático

Durante uma migração com Strangler Fig, o novo serviço de pedidos precisa consultar dados de clientes no legado. Em vez de usar o modelo do legado diretamente (`CUST_TBL.CUST_NM`), a ACL traduz para o modelo novo (`Customer.name`), isolando o novo serviço de mudanças no legado.

## Termos relacionados

- [[strangler-fig-pattern]]
- [[branch-by-abstraction]]

## Fontes

- Eric Evans — "Domain-Driven Design" (2003)
- [[estrategias-modernizacao-sistemas]]
