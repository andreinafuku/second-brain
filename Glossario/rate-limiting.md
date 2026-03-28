---
title: "Rate Limiting"
date: 2026-03-28
tags:
  - glossario
  - arquitetura/resiliencia
  - arquitetura/patterns
area: arquitetura
aliases:
  - Limitação de taxa
  - Throttling
---

# Rate Limiting

## Definição

Mecanismo que controla a quantidade de requests que um cliente pode fazer a um serviço dentro de um período de tempo. Protege sistemas contra abuso, sobrecarga e custos inesperados.

## Contexto de uso

Presente em API Gateways, AI Gateways, serviços públicos e internos. Algoritmos comuns: Token Bucket, Sliding Window, Fixed Window, Leaky Bucket.

## Exemplo prático

Um [[ai-gateway]] configura rate limit de 100 requests/minuto por time. O Time A, que tem 6 devs experimentando com LLMs, não consegue estourar o orçamento de tokens porque o gateway rejeita requests além do limite com HTTP 429.

## Termos relacionados

- [[ai-gateway]]
- [[circuit-breaker]]
- [[backpressure]]

## Fontes

- [[ai-gateway]]
