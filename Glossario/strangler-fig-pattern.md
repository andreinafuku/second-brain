---
title: "Strangler Fig Pattern"
date: 2026-03-28
tags:
  - glossario
  - arquitetura/patterns
  - arquitetura/modernizacao
area: arquitetura
aliases:
  - Strangler Pattern
  - Strangler Fig
---

# Strangler Fig Pattern

## Definição

Pattern de modernização incremental onde um novo sistema cresce ao redor do legado, assumindo funcionalidades gradualmente até que o sistema antigo possa ser desativado. Nomeado por Martin Fowler em referência à figueira estranguladora australiana.

## Contexto de uso

Usado quando se precisa modernizar um sistema legado sem o risco de uma reescrita completa (big bang). Aplicável em migrações de monolito para microsserviços, troca de tecnologia, ou qualquer cenário onde substituição incremental é preferível.

## Exemplo prático

Um e-commerce monolítico precisa migrar para microsserviços. Em vez de reescrever tudo, coloca-se um proxy na frente que roteia `/checkout` para o novo serviço de pagamentos e todo o resto continua no monolito. A cada sprint, mais rotas são migradas.

## Termos relacionados

- [[anti-corruption-layer]]
- [[branch-by-abstraction]]
- [[rate-limiting]]

## Fontes

- [[estrategias-modernizacao-sistemas]]
- Martin Fowler — StranglerFigApplication (2004)
