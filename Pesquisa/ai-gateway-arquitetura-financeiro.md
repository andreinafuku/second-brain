---
title: "AI Gateway Corporativo para Sistema Financeiro"
date: 2026-03-28
tags:
  - pesquisa
  - arquitetura/patterns
  - ai
  - seguranca
  - compliance
  - status/revisado
area: arquitetura
---

# AI Gateway Corporativo para Sistema Financeiro

Desenho arquitetural completo — fluxo de dados e responsabilidades por camada.

**Contexto assumido:** sistema corporativo financeiro, backend em .NET, banco Oracle, necessidade de uso de LLMs externos com forte preocupação com LGPD, [[pii-personally-identifiable-information|PII]], auditoria, observabilidade e redução de acoplamento com provedores.

**Objetivo desta proposta:** definir uma arquitetura moderna de [[ai-gateway]] que permita:
- uso controlado de modelos externos;
- anonimização e pseudonimização antes da saída dos dados;
- roteamento entre múltiplos provedores;
- observabilidade, custo e auditoria;
- separação clara entre regras de negócio, segurança e integração com LLMs.

---

## 📋 1. Resumo executivo

A recomendação é **não permitir que aplicações de negócio chamem LLMs diretamente**.

Em vez disso, toda chamada deve atravessar um **AI Gateway corporativo**, composto por camadas especializadas:

1. **Canal de entrada padronizado** para aplicações internas;
2. **Camada de políticas e classificação** para decidir se a requisição pode ou não sair para LLM externo;
3. **Camada de anonimização / pseudonimização** para tratamento de PII e dados financeiros sensíveis;
4. **Camada de orquestração** para escolher modelo, provedor, fallback e limites;
5. **Camada de execução LLM**, podendo usar o **[[litellm|LiteLLM]]** como gateway técnico multi-provider;
6. **Camada de rastreabilidade e observabilidade**, com logging, métricas, custos e trilha de auditoria;
7. **Camada de reidentificação controlada**, quando o caso de uso exigir devolver dados ao domínio interno.

---

## 🧭 2. Princípios arquiteturais

Esta arquitetura parte dos seguintes princípios:

### 2.1. Zero direct access
Nenhuma aplicação cliente deve chamar OpenAI, Anthropic, Google, OCI ou outro provedor diretamente.

### 2.2. Data minimization
Somente o mínimo necessário de dados deve ser enviado ao modelo.

### 2.3. Policy first
Antes de qualquer chamada ao modelo, a requisição deve ser classificada e validada por políticas.

### 2.4. Provider abstraction
As aplicações não devem conhecer detalhes específicos de cada provedor ou modelo.

### 2.5. Observability by default
Toda chamada deve produzir telemetria, trilha de auditoria e visibilidade de custo.

### 2.6. Reversible exposure only when necessary
Sempre que possível, preferir anonimização irreversível. Quando a reversão for necessária, usar pseudonimização com forte controle.

---

## 🏗️ 3. Visão arquitetural de alto nível

```text
┌─────────────────────────────────────────────────────────────────────┐
│                         Aplicações Corporativas                    │
│  Web / API / Jobs / Assistentes internos / Workflow / Backoffice  │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      API de Entrada do AI Gateway                  │
│      AuthN / AuthZ / rate limit / tenancy / contratos padronizados│
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│            Camada de Classificação e Decisão de Política           │
│  - classifica caso de uso                                           │
│  - identifica criticidade                                           │
│  - decide se pode usar LLM externo, interno ou bloquear             │
└─────────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴────────────────┐
                │                                │
                ▼                                ▼
┌──────────────────────────────┐   ┌──────────────────────────────┐
│ Camada de Sanitização        │   │ Prompt / Content Guardrails  │
│ - PII detection              │   │ - prompt injection checks    │
│ - masking                    │   │ - jailbreak detection        │
│ - pseudonymization           │   │ - output policy rules        │
│ - token vault                │   │ - unsafe content blocking    │
└──────────────────────────────┘   └──────────────────────────────┘
                │                                │
                └───────────────┬────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 Orquestrador de Modelos e Provedores               │
│  - model routing                                                    │
│  - fallback                                                         │
│  - custo x latência x qualidade                                     │
│  - seleção por tipo de tarefa                                       │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 Gateway Técnico Multi-Provider                     │
│                     Exemplo: LiteLLM Proxy                         │
│  - interface única                                                  │
│  - adaptação de payloads                                            │
│  - integração com múltiplos provedores                              │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│     OpenAI / Anthropic / Google / OCI / modelos internos/self-host │
└─────────────────────────────────────────────────────────────────────┘

Fluxos transversais:
- Observabilidade e FinOps
- Auditoria e Compliance
- Catálogo de políticas
- Vault de tokens de pseudonimização
- Gestão de segredos
```

---

## 🧱 4. Proposta de camadas e responsabilidades

### 4.1. Camada 1 — Aplicações corporativas

#### Responsabilidade
Representa os consumidores do AI Gateway:
- frontend;
- APIs de negócio;
- jobs;
- assistentes internos;
- automações;
- processos de suporte operacional.

#### Regras
- nunca embutir chave de provedor de LLM;
- nunca montar integração direta com provedor externo;
- enviar metadados obrigatórios para governança, por exemplo:
  - sistema de origem;
  - caso de uso;
  - domínio de negócio;
  - criticidade;
  - usuário ou serviço chamador;
  - classificação de dados informada pela aplicação.

#### Saída esperada
A aplicação envia uma requisição padronizada ao AI Gateway, por exemplo:
- `taskType`
- `businessDomain`
- `dataClassification`
- `prompt`
- `context`
- `expectedOutputFormat`

---

### 4.2. Camada 2 — API de Entrada do AI Gateway

#### Responsabilidade
É a porta de entrada corporativa para toda chamada de IA.

#### Funções
- autenticação e autorização;
- [[rate-limiting|rate limiting]];
- quotas por aplicação, equipe ou tenant;
- versionamento do contrato da API;
- validação estrutural do payload;
- correlação e rastreabilidade;
- propagação de `correlationId` e `traceId`.

#### Tecnologias possíveis
- ASP.NET Core API;
- API Gateway corporativo já existente;
- Kong, APIM ou equivalente na borda;
- mTLS entre serviços críticos.

#### Observação
Esta camada **não decide política semântica**; ela controla acesso técnico e contrato.

---

### 4.3. Camada 3 — Classificação e Decisão de Política

#### Responsabilidade
Determinar **se** e **como** a requisição pode seguir.

#### Funções
- classificar o caso de uso:
  - sumarização;
  - extração;
  - classificação;
  - geração de texto;
  - consulta assistida;
  - agente com ferramentas;
- identificar domínio:
  - tesouraria;
  - pagamentos;
  - operações financeiras;
- identificar criticidade e sensibilidade;
- decidir:
  - permitido em LLM externo;
  - permitido apenas em modelo interno;
  - permitido apenas com anonimização forte;
  - bloqueado.

#### Regras típicas
Exemplos de política:

```text
Se dominio = "Pagamentos" e payload contém PII + dados bancários
→ exigir pseudonimização antes de sair

Se dominio = "Operações Financeiras" e caso de uso = decisão automatizada
→ bloquear saída para LLM generativo externo

Se caso de uso = classificação simples sem dados pessoais
→ permitir modelo externo econômico
```

#### Implementação recomendada
- motor de políticas declarativas;
- regras configuráveis por domínio;
- versionamento de políticas;
- trilha de auditoria da decisão.

---

### 4.4. Camada 4 — Sanitização, Anonimização e Pseudonimização

#### Responsabilidade
Transformar o payload para reduzir ou eliminar exposição de dados sensíveis.

#### Subcomponentes

##### 4.4.1. Detector de PII
Detecta, por regra e por inferência, elementos como:
- nome;
- CPF;
- CNPJ quando associado a pessoa ou contexto sensível;
- e-mail;
- telefone;
- conta bancária;
- identificadores de cliente;
- endereço;
- valores vinculados a pessoa identificável;
- chaves internas que permitam reidentificação.

##### 4.4.2. Mascarador
Aplica mascaramento parcial quando a semântica ainda precisa ser preservada.

Exemplo:
- `123.456.789-00` → `***.***.***-00`

##### 4.4.3. Pseudonimizador
Substitui entidades por tokens controlados.

Exemplo:
- `João da Silva` → `[[PESSOA_001]]`
- `Conta 12345-6` → `[[CONTA_017]]`

##### 4.4.4. Token Vault
Armazena a tabela de correspondência entre token e valor real, com:
- criptografia forte;
- controle de acesso restrito;
- prazo de retenção;
- auditoria.

#### Estratégias recomendadas
- **anonimização irreversível** quando o resultado não exige reconstrução;
- **pseudonimização reversível** quando o sistema precisa devolver resposta contextual ao usuário final;
- **data minimization** para remover tudo que não agregue valor ao prompt.

#### Ponto crítico
O Gateway deve evitar enviar ao modelo:
- identificadores reais;
- dados bancários completos;
- números de documentos;
- combinações que permitam reidentificação indireta.

---

### 4.5. Camada 5 — Guardrails de prompt e conteúdo

#### Responsabilidade
Impedir abusos, uso indevido e saídas não aceitáveis.

#### Funções
- detecção de prompt injection;
- detecção de jailbreak;
- validação de instruções proibidas;
- bloqueio de vazamento de segredos;
- filtragem de conteúdo sensível na resposta;
- validação do formato esperado de saída.

#### Exemplos
- impedir que o modelo receba instruções para ignorar políticas internas;
- bloquear respostas que tentem reconstruir PII;
- exigir JSON válido para integrações automatizadas;
- aplicar redaction na saída antes de retornar.

---

### 4.6. Camada 6 — Orquestrador de Modelos

#### Responsabilidade
Escolher o melhor modelo para cada tarefa.

#### Critérios de decisão
- custo;
- latência;
- qualidade;
- disponibilidade;
- capacidade de contexto;
- suporte a tool calling;
- região / soberania / compliance;
- classificação do dado.

#### Estratégias
- roteamento por tipo de tarefa;
- fallback por indisponibilidade;
- failover entre provedores;
- canary release de novos modelos;
- A/B test controlado para avaliação.

#### Exemplo de política de roteamento

```text
Classificação simples            → modelo econômico
Sumarização longa                → modelo com contexto maior
Extração estruturada crítica     → modelo mais estável
Casos com dados muito sensíveis  → modelo interno ou região controlada
```

---

### 4.7. Camada 7 — Gateway Técnico Multi-Provider

#### Responsabilidade
Executar a chamada real para o provedor usando interface unificada.

#### Papel do LiteLLM nesta arquitetura
O **[[litellm|LiteLLM]]** encaixa bem aqui como **camada técnica de execução**:
- abstração entre múltiplos provedores;
- payload padronizado;
- fallback e retry;
- compatibilidade com APIs estilo OpenAI;
- desacoplamento básico entre aplicação e modelo.

#### Limite do LiteLLM
Ele não deve concentrar sozinho:
- política corporativa;
- anonimização forte;
- auditoria regulatória completa;
- decisões de compliance.

Por isso, nesta proposta, o LiteLLM fica **abaixo** da camada corporativa de governança.

#### Posição recomendada
```text
Aplicações → AI Gateway Corporativo → LiteLLM → Provedores
```

---

### 4.8. Camada 8 — Provedores de modelo

#### Responsabilidade
Executar inferência.

#### Tipos de destino
- OpenAI;
- Anthropic;
- Google;
- OCI Generative AI;
- modelos internos hospedados pela organização.

#### Recomendação
Manter esta camada substituível. O sistema não deve depender de payloads específicos do provedor fora da camada de execução.

---

### 4.9. Camadas transversais

#### 4.9.1. Observabilidade
Coletar:
- latência total;
- latência por provedor;
- tokens de entrada e saída;
- custo estimado;
- taxa de erro;
- volume por aplicação;
- distribuição por caso de uso;
- versões de política e modelo.

#### 4.9.2. Auditoria
Registrar:
- quem chamou;
- qual caso de uso;
- qual política foi aplicada;
- se houve anonimização;
- qual modelo foi usado;
- se houve fallback;
- hash do prompt sanitizado;
- decisão de bloqueio ou permissão.

#### 4.9.3. Segurança
- gestão de segredos;
- rotação de chaves;
- segregação de ambientes;
- criptografia em trânsito e em repouso;
- princípio do menor privilégio.

#### 4.9.4. FinOps
- orçamento por área;
- limite por aplicação;
- custo por caso de uso;
- alertas de consumo anômalo.

---

## 🔄 5. Fluxo de dados fim a fim

A seguir, um fluxo recomendado para uma chamada típica.

### Passo 1 — Aplicação envia requisição
A aplicação envia:
- prompt;
- contexto;
- metadados de domínio;
- classificação informada;
- identidade do chamador.

### Passo 2 — Entrada técnica
A API do Gateway:
- autentica;
- autoriza;
- valida schema;
- gera `correlationId`.

### Passo 3 — Policy decision
A camada de política:
- classifica o caso de uso;
- decide se a chamada pode sair;
- define o nível de sanitização exigido;
- define categoria de modelo permitida.

### Passo 4 — Sanitização
A camada de anonimização:
- detecta PII;
- remove excesso de contexto;
- mascara ou pseudonimiza dados;
- registra mapa de tokens quando necessário.

### Passo 5 — Guardrails
A camada de segurança semântica:
- verifica prompt injection;
- valida instruções proibidas;
- reforça system prompt corporativo;
- valida formato de saída esperado.

### Passo 6 — Routing
O orquestrador:
- seleciona modelo;
- escolhe provedor;
- aplica fallback policy;
- define timeout e retry.

### Passo 7 — Execução
O LiteLLM ou camada equivalente:
- traduz requisição;
- chama provedor;
- retorna resposta normalizada.

### Passo 8 — Pós-processamento
O Gateway:
- valida a saída;
- remove conteúdo inadequado;
- reidentifica tokens somente quando permitido;
- converte para formato de resposta de negócio.

### Passo 9 — Observabilidade e auditoria
O Gateway grava:
- métricas;
- logs estruturados;
- evento de auditoria;
- custo estimado.

### Passo 10 — Resposta à aplicação
A aplicação recebe somente o conteúdo permitido, já validado e rastreável.

---

## 📊 6. Fluxo ilustrado em diagrama textual

```text
[Aplicação]
   │
   │ 1. Prompt + contexto + metadados
   ▼
[API AI Gateway]
   │
   │ 2. Auth / rate limit / schema validation
   ▼
[Policy Engine]
   │
   ├─ se proibido → [Bloqueio + auditoria]
   │
   └─ se permitido
        ▼
[PII Detection + Sanitization]
        │
        ▼
[Prompt Guardrails]
        │
        ▼
[Model Router]
        │
        ▼
[LiteLLM Proxy]
        │
        ▼
[OpenAI / Anthropic / OCI / Google / Modelo interno]
        │
        ▼
[Post-processing + Output Guardrails]
        │
        ├─ reidentificação controlada, se permitida
        ▼
[Resposta para aplicação]
```

---

## 📋 7. Responsabilidades por camada

| Camada                      | Responsabilidade principal                                 | Não deve fazer                         |
| --------------------------- | ---------------------------------------------------------- | -------------------------------------- |
| Aplicações corporativas     | Solicitar capacidades de IA via contrato padronizado       | Integrar diretamente com provedor      |
| API de entrada              | Auth, quotas, contrato, rastreabilidade                    | Decisão semântica complexa             |
| Policy engine               | Decidir se, como e onde a chamada pode ocorrer             | Executar integração com provedor       |
| Sanitização                 | Detectar PII, mascarar, pseudonimizar                      | Escolher modelo                        |
| Guardrails                  | Proteger prompt e saída                                    | Fazer regra de negócio de domínio      |
| Model router                | Selecionar modelo/provedor/fallback                        | Armazenar segredos de domínio          |
| LiteLLM / execução          | Normalizar chamadas para múltiplos LLMs                    | Governança corporativa completa        |
| Pós-processamento           | Validar saída, reidentificar quando permitido              | Expor conteúdo sem validação           |
| Observabilidade / auditoria | Medir, registrar e provar conformidade                     | Alterar decisão de negócio             |

---

## ⚖️ 8. Decisões de arquitetura recomendadas

### 8.1. Separar governança de execução
A grande decisão desta proposta é separar:
- **governança corporativa**;
- **execução técnica multi-provider**.

Isso evita sobrecarregar uma ferramenta técnica com responsabilidades regulatórias e de negócio.

### 8.2. Usar LiteLLM como camada interna, não como borda principal
O LiteLLM é útil como executor e normalizador, mas a borda corporativa deve permanecer em um gateway sob controle da arquitetura interna.

### 8.3. Externalizar políticas
Políticas não devem ficar hardcoded em controllers ou services. Devem ser configuráveis, versionáveis e auditáveis.

### 8.4. Token vault isolado
O vault de pseudonimização deve ser um componente segregado, com acesso mínimo e auditoria forte.

### 8.5. Preferir resposta estruturada
Sempre que possível, o Gateway deve exigir:
- JSON schema;
- campos bem definidos;
- contratos estáveis para consumo automatizado.

---

## ⚙️ 9. Proposta específica para stack .NET + Oracle

### 9.1. Componentes sugeridos

#### Backend corporativo
- ASP.NET Core para API do AI Gateway;
- middleware para correlação, autenticação e logging;
- services para policy engine, sanitizer, router e post-processing.

#### Persistência
- Oracle para trilha de auditoria, políticas versionadas, catálogos e metadados;
- componente segregado ou armazenamento específico para token vault;
- evitar armazenar prompts crus com PII.

#### Execução LLM
- LiteLLM como proxy técnico multi-provider;
- provedores configurados por ambiente;
- timeout e retry configurados por tarefa.

#### Observabilidade
- OpenTelemetry;
- logs estruturados;
- métricas por rota, modelo, provedor e domínio.

---

## 📦 10. Exemplo de responsabilidades em termos de serviços

```text
AiGateway.Api
  - controllers / endpoints
  - auth / throttling / validation

AiGateway.Policy
  - policy evaluator
  - use case classifier
  - permission matrix

AiGateway.Sanitization
  - pii detector
  - masker
  - pseudonymizer
  - token vault adapter

AiGateway.Guardrails
  - prompt inspector
  - output validator
  - schema enforcer

AiGateway.Routing
  - model selection
  - provider selection
  - fallback strategy

AiGateway.Execution
  - LiteLLM client
  - provider abstraction
  - response normalization

AiGateway.Observability
  - tracing
  - metrics
  - audit writer
  - cost estimator
```

---

## 🎯 11. Casos de uso e decisão arquitetural

### 11.1. Caso de uso: sumarização de atendimento com dados pessoais
#### Decisão
- permitido somente após pseudonimização;
- modelo externo permitido;
- reidentificação opcional apenas no retorno interno.

### 11.2. Caso de uso: classificação de chamados sem PII
#### Decisão
- permitido em modelo econômico;
- sanitização leve;
- alto volume, baixo custo.

### 11.3. Caso de uso: recomendação de decisão financeira individual
#### Decisão
- tende a exigir bloqueio para LLM externo ou uso restrito de modelo interno;
- necessita análise jurídica e regulatória adicional;
- alta exigência de explicabilidade.

### 11.4. Caso de uso: agente com tools acessando sistemas internos
#### Decisão
- exigir sandbox e políticas mais rígidas;
- limitar escopo de tools;
- registrar ação por ação.

---

## ⚠️ 12. Principais riscos e mitigação

### 12.1. Risco: vazamento de PII
#### Mitigação
- PII detection + pseudonimização;
- testes com payloads reais;
- política de mínimo necessário.

### 12.2. Risco: dependência excessiva de um provedor
#### Mitigação
- roteamento multi-provider;
- contratos internos estáveis;
- camada LiteLLM ou equivalente.

### 12.3. Risco: custo fora de controle
#### Mitigação
- quotas;
- budgets;
- roteamento por criticidade;
- métricas por caso de uso.

### 12.4. Risco: ausência de prova de conformidade
#### Mitigação
- auditoria completa;
- versionamento de políticas;
- retenção adequada de evidências.

### 12.5. Risco: respostas não confiáveis
#### Mitigação
- uso preferencial de extração estruturada;
- validação de saída;
- fallback para revisão humana em fluxos críticos.

---

## 🗺️ 13. Roadmap de implantação sugerido

### Fase 1 — Fundação
- criar API do AI Gateway;
- integrar autenticação;
- centralizar chamadas;
- introduzir LiteLLM como executor técnico.

### Fase 2 — Governança mínima viável
- implementar policy engine básico;
- adicionar PII detection inicial;
- registrar auditoria e métricas.

### Fase 3 — Sanitização robusta
- pseudonimização reversível;
- token vault;
- redaction de entrada e saída.

### Fase 4 — Roteamento e FinOps
- múltiplos provedores;
- fallback;
- controle de custo por domínio.

### Fase 5 — Operação madura
- catálogos de modelos;
- políticas avançadas;
- avaliação contínua;
- painéis executivos e operacionais.

---

## ✅ 14. Conclusão

A arquitetura recomendada para um ambiente financeiro corporativo não trata AI Gateway como simples proxy. Ela o trata como uma **plataforma de controle**, responsável por intermediar de forma segura, auditável e economicamente sustentável toda interação entre aplicações internas e modelos de linguagem.

Nesta proposta:
- o **AI Gateway corporativo** concentra governança, política, anonimização, observabilidade e compliance;
- o **LiteLLM** entra como **camada técnica de execução multi-provider**;
- a separação entre **governança** e **execução** reduz acoplamento e aumenta a maturidade arquitetural;
- o desenho permite evolução gradual sem comprometer segurança e LGPD.

---

## 🚀 15. Recomendação final

```text
Aplicações internas
   ↓
AI Gateway corporativo (.NET)
   ↓
Policy Engine + Sanitization + Guardrails + Routing + Audit
   ↓
LiteLLM
   ↓
OpenAI / Anthropic / Google / OCI / modelos internos
```

Esse desenho cria uma base sólida para:
- adoção de múltiplos LLMs;
- proteção de dados financeiros;
- rastreabilidade;
- escalabilidade;
- governança corporativa de IA.

---

## Notas relacionadas

- [[ai-gateway]] — conceito base e responsabilidades do pattern
- [[ai-gateway-implementacoes-modernas]] — padrões de implementação em grandes empresas
- [[litellm]] — ferramenta usada como gateway técnico multi-provider nesta proposta
- [[litellm-como-ai-gateway]] — análise do posicionamento do LiteLLM no espectro de gateways
- [[pii-personally-identifiable-information]] — definição, técnicas de tratamento e relevância para LGPD
- [[rate-limiting]] — mecanismo aplicado na camada de entrada do gateway
