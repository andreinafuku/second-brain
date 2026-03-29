---
title: AI Gateway Corporativo — Desenho Técnico .NET
date: 2026-03-28
tags:
  - arquitetura/ai-gateway
  - pesquisa
  - status/consolidado
---

# AI Gateway Corporativo para Sistema Financeiro
## Desenho técnico com componentes .NET detalhados

**Contexto assumido:** backend em .NET 8, banco Oracle, integração com múltiplos provedores de LLM, necessidade de LGPD, anonimização, auditoria, observabilidade e desacoplamento entre aplicações e modelos.

**Objetivo desta versão:** detalhar uma proposta implementável em .NET, com componentes, responsabilidades, fluxo técnico, contratos, serviços, persistência e preocupações operacionais.

---

## 🏗️ 1. Visão técnica de arquitetura

A arquitetura recomendada separa a solução em dois blocos principais:

### 🏛️ 1.1. Bloco de governança corporativa
Responsável por:
- autenticação e autorização;
- políticas;
- classificação de uso;
- sanitização;
- guardrails;
- auditoria;
- observabilidade;
- pós-processamento.

### ⚙️ 1.2. Bloco de execução multi-provider
Responsável por:
- chamada padronizada aos modelos;
- abstração de provedores;
- fallback e retry;
- compatibilidade entre APIs.

Neste desenho, o bloco de execução pode usar **LiteLLM**.

---

## 🔩 2. Macrocomponentes

```text
┌──────────────────────────────────────────────────────────────────────┐
│                       Consumers / Sistemas internos                  │
│   Frontend, APIs de domínio, jobs, workflows, assistentes internos  │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        AiGateway.Api (.NET 8)                       │
│ Endpoints, auth, quotas, correlation, request validation            │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     Application Orchestration Layer                 │
│ Use cases, command handlers, flow orchestration                     │
└──────────────────────────────────────────────────────────────────────┘
                                 │
         ┌───────────────────────┼────────────────────────┐
         ▼                       ▼                        ▼
┌──────────────────┐   ┌──────────────────────┐   ┌────────────────────┐
│ Policy Services  │   │ Sanitization Services│   │ Guardrail Services │
└──────────────────┘   └──────────────────────┘   └────────────────────┘
         │                       │                        │
         └───────────────────────┼────────────────────────┘
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     Routing / Provider Selection                     │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                  Execution Adapter / LiteLlmClient                  │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│        LiteLLM Proxy / OpenAI / Anthropic / OCI / Google / local    │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│        Post-processing / Output Validation / Reidentification       │
└──────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│   Audit / Metrics / Trace / Cost / Oracle Persistence / Vault       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📁 3. Estrutura de solução .NET sugerida

Uma estrutura possível:

```text
src/
  AiGateway.Api/
  AiGateway.Application/
  AiGateway.Domain/
  AiGateway.Infrastructure/
  AiGateway.Integrations.LiteLlm/
  AiGateway.Integrations.OpenAi/
  AiGateway.Integrations.Anthropic/
  AiGateway.Integrations.OciGenAi/
  AiGateway.Persistence.Oracle/
  AiGateway.Observability/
  AiGateway.Security/
  AiGateway.Contracts/
tests/
  AiGateway.UnitTests/
  AiGateway.IntegrationTests/
  AiGateway.ArchitectureTests/
```

---

## 📦 4. Responsabilidade de cada projeto

### 🌐 4.1. AiGateway.Api
Camada de entrada HTTP.

**Responsabilidades**
- expor endpoints REST;
- autenticar e autorizar;
- validar headers obrigatórios;
- aplicar throttling e rate limit;
- propagar `traceId`, `correlationId`, `tenantId`;
- mapear request/response HTTP para comandos da aplicação.

**Conteúdo típico**
- controllers ou minimal APIs;
- filters;
- middleware;
- model binding;
- configuração de autenticação;
- health checks.

---

### 🎛️ 4.2. AiGateway.Application
Camada de orquestração de casos de uso.

**Responsabilidades**
- coordenar o fluxo completo;
- receber comandos como `ProcessAiRequestCommand`;
- executar policy evaluation;
- chamar serviços de sanitização;
- invocar router e executor;
- registrar eventos de auditoria.

**Conteúdo típico**
- command handlers;
- query handlers;
- DTOs internos;
- interfaces de serviços;
- orquestradores.

> [!warning] Atenção
> Esta camada não deve conter detalhes de HTTP nem detalhes concretos de Oracle, LiteLLM ou OpenAI.

---

### 🧠 4.3. AiGateway.Domain
Camada de modelo de domínio do Gateway.

**Responsabilidades**
- representar conceitos centrais;
- encapsular regras invariantes;
- modelar decisões de política;
- representar classificações, tipos de tarefa e resultado de sanitização.

**Entidades e value objects possíveis**
- `AiRequest`
- `AiResponse`
- `PolicyDecision`
- `SanitizationResult`
- `RoutingDecision`
- `TokenMapping`
- `DataClassification`
- `TaskType`
- `ProviderCapability`

**Regras que podem morar aqui**
- transições válidas de estado;
- coerência entre classificação e política;
- restrições de formato lógico.

---

### ⚙️ 4.4. AiGateway.Infrastructure
Camada com implementações técnicas comuns.

**Responsabilidades**
- clock;
- GUID generator;
- serialization;
- encryption services;
- cache;
- configuration providers;
- adapters compartilhados.

---

### 🗄️ 4.5. AiGateway.Persistence.Oracle
Persistência Oracle.

**Responsabilidades**
- gravação de auditoria;
- armazenamento de políticas versionadas;
- catálogos de modelos e provedores;
- métricas agregadas;
- tabelas operacionais do Gateway.

**Possíveis tabelas**
- `AI_REQUEST_AUDIT`
- `AI_POLICY_DECISION`
- `AI_PROVIDER_CATALOG`
- `AI_MODEL_CATALOG`
- `AI_COST_USAGE`
- `AI_PROMPT_HASH_LOG`
- `AI_TOKEN_REFERENCE` (somente se fizer sentido armazenar metadados de tokenização fora do vault principal)

> [!warning] Segurança
> - Evitar armazenar prompt bruto com PII;
> - preferir hashes, trechos redigidos ou payloads sanitizados;
> - usar criptografia ou coluna protegida quando necessário.

---

### 🔌 4.6. AiGateway.Integrations.LiteLlm
Cliente técnico para o proxy LiteLLM.

**Responsabilidades**
- montar payload compatível;
- chamar endpoint do LiteLLM;
- mapear resposta para contrato interno;
- tratar timeout, retry e erro técnico.

**Interface sugerida**
```csharp
public interface ILlmExecutionClient
{
    Task<LlmExecutionResult> ExecuteAsync(LlmExecutionRequest request, CancellationToken cancellationToken);
}
```

**Implementação**
`LiteLlmExecutionClient : ILlmExecutionClient`

---

### 🔐 4.7. AiGateway.Security
Serviços de segurança e sanitização.

**Responsabilidades**
- detecção de PII;
- mascaramento;
- pseudonimização;
- criptografia;
- integração com vault;
- output redaction.

**Serviços possíveis**
- `IPiiDetector`
- `IDataMaskingService`
- `IPseudonymizationService`
- `ITokenVaultService`
- `IOutputRedactionService`

---

### 📊 4.8. AiGateway.Observability
Telemetria.

**Responsabilidades**
- OpenTelemetry tracing;
- métricas customizadas;
- logging estruturado;
- exportação para plataforma de monitoramento.

**Métricas sugeridas**
- requisições por aplicação;
- latência total;
- latência por provedor;
- tokens de entrada e saída;
- custo estimado;
- falhas por tipo;
- bloqueios por política.

---

## 🔄 5. Pipeline técnico da requisição

A seguir, o pipeline implementável.

### 📥 5.1. Recepção da chamada
A API recebe um payload padronizado.

Exemplo lógico:
```json
{
  "taskType": "summarization",
  "businessDomain": "payments",
  "dataClassification": "restricted",
  "userContext": {
    "userId": "abc",
    "sourceSystem": "PortalFinanceiro"
  },
  "prompt": "Resumo do histórico do cliente ...",
  "contextItems": []
}
```

### ✅ 5.2. Validação técnica
Executar:
- schema validation;
- autenticação;
- autorização;
- quotas;
- geração de `correlationId`.

### ⚖️ 5.3. Policy evaluation
`IPolicyEvaluator` devolve algo como:
- permitido ou bloqueado;
- nível de sanitização;
- categoria de modelos permitida;
- exigência de output estruturado;
- flags de auditoria reforçada.

### 🧹 5.4. Sanitização
`ISanitizationPipeline` executa:
- PII detection;
- minimização do contexto;
- masking;
- pseudonimização;
- persistência de token mapping em vault.

### 🛡️ 5.5. Guardrails de entrada
`IPromptGuardService`:
- detecta prompt injection;
- adiciona instruções corporativas;
- valida conteúdo proibido;
- reforça formato de saída esperado.

### 🔀 5.6. Routing
`IModelRouter` escolhe:
- provedor;
- modelo;
- temperatura;
- timeout;
- fallback chain.

### 🚀 5.7. Execução
`ILlmExecutionClient` chama LiteLLM.

### 📤 5.8. Pós-processamento
`IOutputProcessor`:
- valida JSON schema;
- remove conteúdo inadequado;
- reidentifica tokens quando permitido;
- gera resposta final.

### 📝 5.9. Auditoria e observabilidade
Executar:
- audit write;
- metric publish;
- tracing spans;
- cálculo de custo estimado.

---

## 📐 6. Interfaces centrais sugeridas

### 6.1. Avaliação de política
```csharp
public interface IPolicyEvaluator
{
    Task<PolicyDecision> EvaluateAsync(AiRequest request, CancellationToken cancellationToken);
}
```

### 6.2. Pipeline de sanitização
```csharp
public interface ISanitizationPipeline
{
    Task<SanitizationResult> SanitizeAsync(
        AiRequest request,
        PolicyDecision decision,
        CancellationToken cancellationToken);
}
```

### 6.3. Guardrails de entrada
```csharp
public interface IPromptGuardService
{
    Task<PromptGuardResult> InspectAsync(
        SanitizedPrompt prompt,
        PolicyDecision decision,
        CancellationToken cancellationToken);
}
```

### 6.4. Router
```csharp
public interface IModelRouter
{
    Task<RoutingDecision> RouteAsync(
        AiRequest request,
        PolicyDecision policyDecision,
        SanitizationResult sanitizationResult,
        CancellationToken cancellationToken);
}
```

### 6.5. Pós-processamento
```csharp
public interface IOutputProcessor
{
    Task<AiResponse> ProcessAsync(
        RawLlmResponse rawResponse,
        PolicyDecision policyDecision,
        SanitizationResult sanitizationResult,
        CancellationToken cancellationToken);
}
```

---

## 📋 7. Contratos internos recomendados

### 7.1. AiRequest
```csharp
public sealed class AiRequest
{
    public required string CorrelationId { get; init; }
    public required string SourceSystem { get; init; }
    public required string BusinessDomain { get; init; }
    public required TaskType TaskType { get; init; }
    public required DataClassification DataClassification { get; init; }
    public required string Prompt { get; init; }
    public IReadOnlyCollection<ContextItem> ContextItems { get; init; } = [];
    public bool RequiresStructuredOutput { get; init; }
}
```

### 7.2. PolicyDecision
```csharp
public sealed class PolicyDecision
{
    public required bool IsAllowed { get; init; }
    public required string PolicyVersion { get; init; }
    public required SanitizationLevel SanitizationLevel { get; init; }
    public required AllowedModelProfile AllowedModelProfile { get; init; }
    public required bool AllowReidentification { get; init; }
    public string? BlockReason { get; init; }
}
```

### 7.3. RoutingDecision
```csharp
public sealed class RoutingDecision
{
    public required string Provider { get; init; }
    public required string Model { get; init; }
    public required TimeSpan Timeout { get; init; }
    public required IReadOnlyCollection<FallbackTarget> FallbackTargets { get; init; }
}
```

---

## 🧩 8. Componentes de domínio e aplicação

### ⚖️ 8.1. Policy Engine
Deve permitir regras por:
- domínio de negócio;
- classificação do dado;
- tipo de tarefa;
- origem da chamada;
- ambiente;
- exigência regulatória.

**Implementação possível**
- regras declarativas em banco;
- carregamento em cache;
- avaliação por motor interno;
- versionamento por vigência.

---

### 🧹 8.2. Sanitization Pipeline
Pode ser organizado em steps:

```text
1. Context minimizer
2. PII detector
3. Masking strategy
4. Pseudonymization strategy
5. Token vault write
6. Sanitized payload builder
```

**Estratégia prática**

Criar uma interface de step:
```csharp
public interface ISanitizationStep
{
    Task ApplyAsync(SanitizationContext context, CancellationToken cancellationToken);
}
```

E então compor um pipeline.

---

### 🛡️ 8.3. Guardrails
Separar guardrails de entrada e de saída.

**Entrada**
- prompt injection;
- secrets leakage prevention;
- conteúdo proibido;
- policy hints.

**Saída**
- remoção de dados proibidos;
- validação de schema;
- bloqueio de linguagem não permitida em integrações críticas;
- redaction adicional.

---

### 🔀 8.4. Router
O router pode usar:
- capability catalog;
- health status;
- custo por 1k tokens;
- perfil do caso de uso.

**Exemplo de perfis**
- `LowCostClassification`
- `BalancedSummarization`
- `HighReliabilityStructuredExtraction`
- `InternalOnlySensitiveCase`

---

## 💻 9. Exemplo de fluxo em código de aplicação

```csharp
public sealed class ProcessAiRequestHandler
{
    private readonly IPolicyEvaluator _policyEvaluator;
    private readonly ISanitizationPipeline _sanitizationPipeline;
    private readonly IPromptGuardService _promptGuardService;
    private readonly IModelRouter _modelRouter;
    private readonly ILlmExecutionClient _llmExecutionClient;
    private readonly IOutputProcessor _outputProcessor;
    private readonly IAuditService _auditService;

    public async Task<AiResponse> HandleAsync(AiRequest request, CancellationToken ct)
    {
        var policyDecision = await _policyEvaluator.EvaluateAsync(request, ct);

        if (!policyDecision.IsAllowed)
        {
            await _auditService.WriteBlockedAsync(request, policyDecision, ct);
            throw new InvalidOperationException(policyDecision.BlockReason ?? "Request blocked");
        }

        var sanitizationResult = await _sanitizationPipeline.SanitizeAsync(request, policyDecision, ct);

        var promptGuardResult = await _promptGuardService.InspectAsync(
            sanitizationResult.SanitizedPrompt,
            policyDecision,
            ct);

        if (!promptGuardResult.IsAllowed)
        {
            await _auditService.WriteGuardrailBlockAsync(request, promptGuardResult, ct);
            throw new InvalidOperationException(promptGuardResult.BlockReason ?? "Guardrail blocked");
        }

        var routingDecision = await _modelRouter.RouteAsync(request, policyDecision, sanitizationResult, ct);

        var rawResponse = await _llmExecutionClient.ExecuteAsync(
            new LlmExecutionRequest(
                routingDecision.Provider,
                routingDecision.Model,
                promptGuardResult.FinalPrompt),
            ct);

        var response = await _outputProcessor.ProcessAsync(rawResponse, policyDecision, sanitizationResult, ct);

        await _auditService.WriteSuccessAsync(request, policyDecision, routingDecision, response, ct);

        return response;
    }
}
```

---

## 🗄️ 10. Persistência Oracle detalhada

### 📝 10.1. Auditoria
Tabela sugerida `AI_REQUEST_AUDIT`

Campos possíveis:
- `ID`
- `CORRELATION_ID`
- `SOURCE_SYSTEM`
- `BUSINESS_DOMAIN`
- `TASK_TYPE`
- `DATA_CLASSIFICATION`
- `REQUEST_TIMESTAMP`
- `POLICY_VERSION`
- `POLICY_RESULT`
- `PROVIDER_USED`
- `MODEL_USED`
- `INPUT_TOKEN_COUNT`
- `OUTPUT_TOKEN_COUNT`
- `ESTIMATED_COST`
- `HAS_PII_DETECTED`
- `SANITIZATION_LEVEL`
- `RESPONSE_STATUS`
- `PROMPT_HASH`

### ⚖️ 10.2. Política
Tabela sugerida `AI_POLICY_RULE`

Campos possíveis:
- `RULE_ID`
- `RULE_NAME`
- `BUSINESS_DOMAIN`
- `TASK_TYPE`
- `DATA_CLASSIFICATION`
- `ALLOW_EXTERNAL_LLM`
- `REQUIRED_SANITIZATION_LEVEL`
- `ALLOWED_MODEL_PROFILE`
- `ALLOW_REIDENTIFICATION`
- `VERSION`
- `ACTIVE_FLAG`

### 🤖 10.3. Catálogo de modelos
Tabela sugerida `AI_MODEL_CATALOG`

Campos possíveis:
- `MODEL_ID`
- `PROVIDER_NAME`
- `MODEL_NAME`
- `PROFILE_NAME`
- `MAX_CONTEXT_TOKENS`
- `SUPPORTS_TOOLS`
- `AVG_COST_INPUT`
- `AVG_COST_OUTPUT`
- `ACTIVE_FLAG`

---

## 🔗 11. Integração com LiteLLM

### 11.1. Papel
O LiteLLM fica atrás do Gateway corporativo, como executor técnico.

### 11.2. Benefícios
- simplifica integração com múltiplos provedores;
- reduz necessidade de múltiplos SDKs;
- facilita fallback técnico;
- ajuda em padronização de payload.

### 11.3. Cuidados
- não colocar regras de negócio no LiteLLM;
- não depender dele para anonimização;
- não delegar compliance corporativo a ele.

### 11.4. Exemplo de adapter
```csharp
public sealed class LiteLlmExecutionClient : ILlmExecutionClient
{
    private readonly HttpClient _httpClient;

    public LiteLlmExecutionClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<LlmExecutionResult> ExecuteAsync(LlmExecutionRequest request, CancellationToken cancellationToken)
    {
        // montar payload
        // chamar endpoint do LiteLLM
        // tratar erros
        // mapear tokens, finish reason e conteúdo
        throw new NotImplementedException();
    }
}
```

---

## 🔧 12. Middleware e aspectos transversais

### 12.1. Middleware HTTP sugeridos
- correlation middleware;
- authentication middleware;
- structured logging middleware;
- exception handling middleware;
- tenant resolution middleware.

### 12.2. Telemetria
Criar spans como:
- `ai.request.received`
- `ai.policy.evaluated`
- `ai.sanitization.completed`
- `ai.routing.completed`
- `ai.execution.completed`
- `ai.output.processed`

### 12.3. Logging estruturado
Sempre registrar:
- `correlationId`
- `sourceSystem`
- `businessDomain`
- `taskType`
- `provider`
- `model`
- `policyVersion`
- `sanitizationLevel`

> [!warning] Segurança
> Sem registrar payload sensível bruto.

---

## 🔒 13. Segurança detalhada

### 13.1. Gestão de segredos
- chaves de provedores fora do código;
- usar secret store;
- rotação periódica;
- segregação por ambiente.

### 13.2. Vault de pseudonimização
Separar do banco operacional quando possível.

**Responsabilidades**
- gerar token;
- armazenar mapeamento;
- permitir reidentificação controlada;
- registrar acessos.

### 13.3. Criptografia
- TLS em trânsito;
- criptografia em repouso;
- proteção especial para mappings reversíveis.

---

## 🧪 14. Estratégia de testes

### 14.1. Unit tests
Cobrir:
- policy evaluator;
- sanitization steps;
- routing rules;
- output validator.

### 14.2. Integration tests
Cobrir:
- API → aplicação → persistência Oracle;
- API → LiteLLM mock;
- auditoria;
- tracing.

### 14.3. Contract tests
Garantir contrato estável:
- requests da aplicação;
- integração com LiteLLM;
- estrutura de resposta.

### 14.4. Security tests
- payloads com PII realista;
- prompt injection;
- tentativas de jailbreak;
- reidentificação indevida.

---

## 🚀 15. Estratégia de deploy

### 15.1. Serviços
Possível divisão:
- `ai-gateway-api`
- `ai-policy-service`
- `ai-sanitization-service`
- `ai-routing-service`
- `ai-audit-worker`

Ou começar como modular monolith em .NET e evoluir depois.

### 15.2. Recomendação prática

> [!tip] Recomendação
> Para o cenário corporativo, começar com **modular monolith bem separado** costuma ser mais eficiente:
> - menor custo operacional;
> - menor complexidade distribuída;
> - mais facilidade de evolução inicial.
>
> Depois, extrair componentes de maior criticidade ou escala.

---

## 🗺️ 16. Roadmap técnico sugerido

### Fase 1
- API padronizada;
- contrato interno;
- integração com LiteLLM;
- auditoria mínima;
- correlação e tracing.

### Fase 2
- policy engine inicial;
- catálogo de modelos;
- roteamento por perfil;
- quotas e budgets.

### Fase 3
- PII detector;
- masking;
- pseudonimização;
- vault.

### Fase 4
- guardrails robustos;
- output validation;
- JSON schema enforcement;
- fallback avançado.

### Fase 5
- painéis operacionais;
- avaliação contínua de qualidade;
- A/B testing;
- governança madura.

---

## 🧠 18. AI Gateway vs AI Agents — Separação conceitual

> [!question] Dúvida frequente
> Este documento propõe a implementação de agentes de IA em .NET, ou apenas a implementação do AI Gateway?

A resposta direta é: **o documento descreve exclusivamente o AI Gateway** — não a implementação de agentes de IA.

Esse desenho é de infraestrutura de controle e orquestração, não de comportamento autônomo. É importante entender a separação entre esses dois conceitos.

### 🧱 18.1. AI Gateway (o que este documento descreve)

Middleware corporativo com foco em:
- segurança (PII, LGPD);
- governança;
- roteamento de modelos;
- observabilidade;
- controle de custo.

> [!info] Característica essencial
> O Gateway **não pensa, não decide sobre negócio**, não executa planos e não usa ferramentas. É infraestrutura.

### 🤖 18.2. AI Agents (outra camada, fora deste escopo)

Sistemas que:
- planejam e tomam decisões;
- usam ferramentas (tools);
- executam múltiplos passos;
- podem chamar APIs, iterar e manter estado;
- agem de forma semi-autônoma.

> [!info] Característica essencial
> Agents são **comportamento inteligente**, não infraestrutura.

---

### 📐 18.3. Como os dois se encaixam na arquitetura

A arquitetura moderna posiciona os dois em camadas distintas:

```text
[ Aplicações / Agents ]
         ↓
   [ AI Gateway ]
         ↓
      [ LLMs ]
```

**Agents consomem o AI Gateway. O Gateway governa o acesso aos LLMs.**

> [!warning] Erro comum
> Misturar lógica de agent dentro do Gateway é um equívoco arquitetural. O resultado é:
> - código difícil de manter;
> - mistura de domínio com infraestrutura;
> - quebra de governança;
> - dificuldade de auditoria.

---

### 🏦 18.4. Onde agentes entram no cenário financeiro

Em um sistema financeiro, agentes seriam elementos como:
- assistente de conciliação financeira;
- análise de inconsistências;
- recomendação de ação operacional;
- automação de atendimento interno.

Esses agentes ficam **fora do Gateway** e usam o Gateway como canal controlado.

---

### 💻 18.5. .NET vs Python para agentes

**Python é hoje a linguagem mais produtiva para agentes**, principalmente por:
- ecossistema mais maduro: LangChain, LlamaIndex, AutoGen;
- velocidade de experimentação;
- maior disponibilidade de exemplos e bibliotecas.

**.NET não está fora do jogo.** É possível implementar agentes com:
- **Semantic Kernel**
- **Microsoft.Extensions.AI**

Porém o ecossistema ainda é menor que Python para casos de agentes complexos.

---

### 🧭 18.6. Recomendação arquitetural para o cenário .NET + Oracle

A abordagem recomendada é **híbrida**:

| Camada | Linguagem | Justificativa |
|--------|-----------|---------------|
| AI Gateway | **.NET** | Governança, segurança, LGPD, integração corporativa, performance previsível |
| AI Agents | **Python** | Velocidade de evolução, frameworks mais maduros, experimentação |

A integração entre as camadas ocorre via:

```text
Agent (Python)
     ↓
AI Gateway (.NET)
     ↓
LLMs
```

Comunicação via **HTTP (REST)** ou **gRPC**.

> [!tip] Insight para ambientes maduros
> Em empresas com governança consolidada, o agent **nunca fala diretamente com OpenAI ou outro provedor**. O fluxo obrigatório é sempre:
>
> `Agent → Gateway → LLM`
>
> Mesmo que o agent seja Python.

---

## 🏁 17. Recomendação técnica final

Para o cenário .NET + Oracle, a recomendação é:

```text
Sistemas internos
   ↓
AiGateway.Api (.NET 8)
   ↓
AiGateway.Application
   ├─ PolicyEvaluator
   ├─ SanitizationPipeline
   ├─ PromptGuardService
   ├─ ModelRouter
   ├─ OutputProcessor
   └─ AuditService
   ↓
LiteLlmExecutionClient
   ↓
LiteLLM
   ↓
OpenAI / Anthropic / Google / OCI / modelos internos
   ↓
Oracle + Vault + Observability stack
```

Esse desenho preserva:
- separação de responsabilidades;
- evolutividade;
- governança;
- aderência à LGPD;
- flexibilidade multi-provider;
- viabilidade de implementação em stack .NET corporativa.
