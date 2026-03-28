---
title: "LiteLLM - Documentação Oficial"
date: 2026-03-28
tags:
  - referencia
  - ai
  - ferramentas
  - status/revisado
tipo: documentação
autor: BerriAI
url: https://docs.litellm.ai/docs/
---

# LiteLLM - Documentação Oficial

## Dados da fonte

| Campo  | Valor                             |
| ------ | --------------------------------- |
| Tipo   | Documentação oficial              |
| Autor  | BerriAI                           |
| URL    | https://docs.litellm.ai/docs/     |
| GitHub | https://github.com/BerriAI/litellm |
| Acesso | 2026-03-28                        |

## Resumo

Documentação completa do LiteLLM, biblioteca open source que oferece interface unificada (formato OpenAI) para 100+ LLMs. Cobre o SDK Python, o Proxy Server (LLM Gateway), routing, virtual keys, spend tracking e integrações de observabilidade.

## Pontos-chave

- Dois modos de uso: SDK Python (embutido na aplicação) e Proxy Server (gateway auto-hospedado)
- Router com load balancing, fallback, retry e cooldown entre deployments
- Virtual keys com controle de acesso e budgets por chave/usuário/time
- Spend tracking automático com cálculo de custo por modelo (PostgreSQL)
- Integrações de observabilidade: Langfuse, MLflow, Helicone, Lunary
- Compatível com OpenAI SDK, Anthropic SDK, Langchain, LlamaIndex e qualquer cliente OpenAI-compatible
- Suporta MCP e agents como gateway unificado

## Citações relevantes

> "LiteLLM is an open-source library that gives you a single, unified interface to call 100+ LLMs — OpenAI, Anthropic, Vertex AI, Bedrock, and more — using the OpenAI format."

> "LiteLLM is a unified gateway for LLMs, agents, and MCP — you don't need a separate agent or MCP gateway."

## Como se conecta ao meu contexto

Fonte primária para o estudo de [[litellm]]. Documentação bem estruturada com quick starts, tutoriais e referência de API. Útil para avaliar o LiteLLM como implementação do pattern [[ai-gateway]].

## Notas relacionadas

- [[litellm]]
- [[ai-gateway]]
- [[referencia-portkey-ai-gateway]]
