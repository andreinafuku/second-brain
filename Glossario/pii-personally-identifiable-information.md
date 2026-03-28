---
title: PII - Personally Identifiable Information
date: 2026-03-28
tags:
  - glossario
  - seguranca
  - compliance
  - status/revisado
  - pii
area: arquitetura
aliases:
  - PII
  - Informação Pessoal Identificável
  - Dado Pessoal
---

# PII - Personally Identifiable Information

**PII** significa **Personally Identifiable Information** (em português: **Informação Pessoal Identificável**).

Trata-se de qualquer dado que possa identificar direta ou indiretamente uma pessoa física.

## 📌 Definição

PII é **qualquer informação que permita identificar um indivíduo**, isoladamente ou quando combinada com outros dados.

## 🔎 Contexto de uso

PII está presente em qualquer sistema que processe dados de pessoas físicas. No Brasil, é regulado pela **LGPD** (Lei Geral de Proteção de Dados Pessoais — Lei nº 13.709/2018).

### 🔴 Identificadores diretos

- Nome completo
- CPF / RG
- Número de passaporte
- E-mail pessoal
- Telefone
- Endereço residencial

### 🟠 Identificadores indiretos (quase-identificadores)

- Data de nascimento
- Endereço IP
- Geolocalização
- Cargo + empresa
- Histórico de transações
- Dados de comportamento

> [!warning] Combinação de dados
> Identificadores indiretos, isoladamente, podem não identificar ninguém — mas **em conjunto**, sim. Esse é um dos principais vetores de re-identificação.

### 🔥 Dados pessoais sensíveis (LGPD)

Subconjunto com nível ainda mais alto de proteção e exigências legais específicas:

- Origem racial/étnica
- Religião
- Saúde
- Dados biométricos
- Vida sexual

## 🧠 Por que PII é crítico em IA (LLMs, AI Gateway)

### 1. 🔐 Privacidade e conformidade legal

No Brasil, a LGPD regula o uso de PII:

- Uso indevido pode gerar multas e sanções
- Necessidade de base legal para tratamento (consentimento, legítimo interesse, etc.)

### 2. ⚠️ Risco de vazamento via LLMs

Ao enviar PII para modelos externos (OpenAI, Anthropic, etc.):

- Pode haver **retenção em logs** pelo provedor
- Pode haver **exposição indireta** via fine-tuning ou caching
- Pode violar políticas internas ou regulatórias

### 3. 🧱 Papel do AI Gateway

O [[ai-gateway]] é a camada responsável por interceptar e tratar PII antes que chegue aos provedores externos:

- **Detecção de PII** no conteúdo do prompt
- **Mascaramento / anonimização** antes de enviar
- **Tokenização** (reversível ou não)
- **Auditoria e rastreabilidade** de todos os acessos

## 🔹 Exemplo prático

Em um sistema financeiro com integração a LLMs externos, dados como contas, transações, CPF e valores vinculados a pessoas são PII. O fluxo correto via AI Gateway:

1. Aplicação envia prompt com dados do usuário
2. Gateway detecta e substitui PII por tokens (`CPF_TOKEN_42`)
3. Prompt sanitizado é enviado ao LLM externo
4. Resposta retorna com tokens
5. Gateway restaura os dados originais (quando necessário) antes de devolver à aplicação

## 🛠️ Técnicas de tratamento de PII

| Técnica                  | Reversível | Exemplo                                    |
| ------------------------ | ---------- | ------------------------------------------ |
| Mascaramento             | Não        | `123.456.789-00` → `***.***.**-**`         |
| Anonimização             | Não        | `João Silva` → `[USER_123]`                |
| Pseudonimização          | Sim        | Cliente → ID interno criptografado         |
| Redação automática (NLP) | Não        | LLM ou regex remove/substitui entidades    |

## 📊 Classificação: nem toda PII é igual

A LGPD distingue dados pessoais comuns de dados pessoais **sensíveis** — estes últimos exigem base legal específica e controles adicionais (listados na seção acima).

## Termos relacionados

- [[ai-gateway]] — responsável pela detecção e mascaramento de PII antes de chamadas a LLMs
- [[litellm]] — gateway open source com anonimização básica, sem policy engine avançado
- [[litellm-como-ai-gateway]] — análise das limitações de PII do LiteLLM vs. gateways enterprise
- [[ai-gateway-implementacoes-modernas]] — Padrão 3: Policy Enforcement Layer

## Fontes

- Lei Geral de Proteção de Dados Pessoais — Lei nº 13.709/2018
- [[ai-gateway-implementacoes-modernas]]
