---
title: "AI Gateway"
date: 2026-03-28
tags:
  - pesquisa
  - arquitetura/patterns
  - ai
  - status/revisado
area: arquitetura
---

# AI Gateway

## Contexto

Com a adoção crescente de LLMs em serviços internos, surge a necessidade de um componente centralizado que intermedie a comunicação entre aplicações e provedores de modelos externos. Estudo focado em entender o pattern, responsabilidades e quando adotar.

## Resumo

AI Gateway é um componente arquitetural que atua como proxy/mediador entre serviços internos e APIs de modelos LLM externos (OpenAI, Anthropic, etc). Centraliza preocupações transversais como segurança, rate limiting, observabilidade e roteamento, evitando que cada serviço implemente essas responsabilidades individualmente.

## Pontos principais

- **Posicionamento**: fica entre a aplicação (consumer) e os provedores de LLM (providers)
- **Rate limiting**: controle de uso por serviço/time/usuário para evitar custos descontrolados
- **Routing**: direcionar requests para diferentes modelos/provedores baseado em custo, latência ou capacidade
- **Anonymization**: remover/mascarar PII antes de enviar dados para APIs externas
- **Security**: autenticação centralizada, controle de acesso, auditoria de uso
- **Observability**: logging, métricas de custo, latência e tokens consumidos
- **Caching**: respostas idênticas podem ser cacheadas para reduzir custo e latência
- **Fallback**: retry automático e failover entre provedores

## Como aplicar

- Avaliar se nossos serviços já consomem LLMs de forma descentralizada
- Propor como componente de plataforma se houver mais de 2 serviços consumindo LLMs
- Pode ser implementado como sidecar, API gateway dedicado ou biblioteca compartilhada
- Discutir com o time a abordagem mais adequada ao nosso contexto

## Diagramas / Visualizações

```mermaid
graph LR
    A[Serviço A] --> GW[AI Gateway]
    B[Serviço B] --> GW
    C[Serviço C] --> GW
    GW -->|Rate Limit| RL[Rate Limiter]
    GW -->|Anonymize| AN[PII Filter]
    GW -->|Route| RT[Router]
    RT --> O[OpenAI]
    RT --> AN2[Anthropic]
    RT --> L[LLM Local]
```

```mermaid
sequenceDiagram
    participant App as Aplicação
    participant GW as AI Gateway
    participant Cache as Cache
    participant LLM as LLM Provider

    App->>GW: Request (prompt + context)
    GW->>GW: Autenticação / Rate limit
    GW->>GW: Anonymization (PII)
    GW->>Cache: Verifica cache
    alt Cache hit
        Cache-->>GW: Resposta cacheada
    else Cache miss
        GW->>LLM: Forward request
        LLM-->>GW: Response
        GW->>Cache: Salva no cache
    end
    GW-->>App: Response
```

## Perguntas em aberto

- Qual o overhead de latência aceitável para o gateway?
- Como lidar com streaming responses (SSE) no gateway?
- Qual estratégia de cache faz sentido para respostas não-determinísticas?

## Fontes

- [[referencia-aws-ai-gateway]]
- [[referencia-portkey-ai-gateway]]

## Notas relacionadas

- [[strangler-fig-pattern]] — pode ser útil para migrar serviços existentes para usar o gateway gradualmente
- [[rate-limiting]] — termo no glossário
