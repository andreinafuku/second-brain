---
title: Agente Financeiro Real para Análise de Divergências em Pagamentos
date: 2026-03-28
tags:
  - arquitetura/ai-agents
  - arquitetura/ai-gateway
  - pesquisa
  - status/consolidado
---

# Agente Financeiro Real para Análise de Divergências em Pagamentos
## Desenho arquitetural completo, fluxo operacional, tools, guardrails e integração com AI Gateway

**Contexto assumido:** sistema financeiro corporativo, backend principal em .NET, banco Oracle, necessidade de usar LLMs externos com controle de LGPD, PII, auditoria, observabilidade e baixo acoplamento com provedores.

---

## 🎯 1. Objetivo do documento

Este documento propõe o desenho de um **agente financeiro real** para um cenário corporativo: investigação de divergências em pagamentos. O agente não substitui sistemas transacionais nem executa decisões críticas de forma autônoma. Ele atua como uma camada de análise assistida, capaz de:

- interpretar uma solicitação do usuário;
- consultar múltiplas fontes internas por meio de tools;
- consolidar evidências;
- montar uma hipótese explicativa;
- sugerir próximos passos;
- encaminhar o caso para revisão humana quando necessário;
- consumir LLMs exclusivamente por meio de um **AI Gateway corporativo**.

A proposta separa claramente:
- **Agent Service**: responsável por planejamento, uso de tools, estado e interação;
- **AI Gateway**: responsável por políticas, anonimização, roteamento, auditoria e governança;
- **Sistemas internos**: responsáveis pelos dados de negócio e pelas ações operacionais.

---

## 📋 2. Caso de uso escolhido

### 🏷️ 2.1. Nome do agente
**Agente de Análise de Divergências em Pagamentos**

### ❓ 2.2. Perguntas que ele deve responder
Exemplos:
- "Por que o pagamento do lote 8457 ficou divergente ontem?"
- "Quais pagamentos rejeitados da concessionária X precisam de ação hoje?"
- "Resuma a causa provável das divergências do fechamento de ontem."
- "Quais evidências indicam falha de integração, regra de negócio ou dado inconsistente?"

### ✅ 2.3. Resultado esperado
O agente deve devolver:
- resumo executivo do caso;
- evidências encontradas;
- classificação da provável causa;
- nível de confiança;
- recomendações de ação;
- indicação se o caso exige revisão humana;
- referência dos registros consultados.

---

## 🔍 3. O que esse agente é — e o que ele não é

### ✔️ 3.1. O que ele é
- um **agente operacional assistivo**;
- capaz de usar tools internas;
- capaz de planejar em múltiplas etapas;
- com memória de execução;
- com trilha de auditoria;
- com saída estruturada.

### ✖️ 3.2. O que ele não é
- não substitui motor de regras;
- não faz liquidação financeira;
- não aprova pagamentos sozinho;
- não escreve diretamente em sistemas críticos sem aprovação;
- não chama OpenAI, Anthropic ou outro provedor sem passar pelo Gateway.

---

## 🧱 4. Princípios arquiteturais

1. **Gateway obrigatório**: o agente nunca chama LLM externo diretamente.
2. **Tool-first para dados**: o agente obtém fatos por tools internas, e não por memória do modelo.
3. **Workflow antes de autonomia total**: começar com fluxo controlado, com pontos determinísticos.
4. **Structured output**: o agente deve produzir JSON validável para consumo seguro.
5. **Human-in-the-loop em casos sensíveis**: ações críticas e casos ambíguos exigem aprovação.
6. **Observabilidade de ponta a ponta**: rastrear tool calls, handoffs, prompts sanitizados e decisões.
7. **Menor contexto necessário**: enviar ao modelo apenas o mínimo necessário.

---

## 🏗️ 5. Arquitetura proposta

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        Usuário / Portal Interno                    │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    API de Conversa / Agent Facade                  │
│  - autenticação do usuário                                          │
│  - sessão                                                           │
│  - streaming de resposta                                            │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Agent Service (Python)                          │
│  - planner/orchestrator                                             │
│  - estado de execução                                               │
│  - uso de tools                                                     │
│  - validação de confiança                                           │
│  - escalation / human review                                        │
└─────────────────────────────────────────────────────────────────────┘
        │                     │                         │
        │                     │                         │
        ▼                     ▼                         ▼
┌──────────────────┐  ┌──────────────────────┐  ┌────────────────────┐
│ Tool: Pagamentos │  │ Tool: Títulos/Boletos│  │ Tool: Eventos/Logs │
└──────────────────┘  └──────────────────────┘  └────────────────────┘
        │                     │                         │
        └─────────────────────┴───────────────┬─────────┘
                                              ▼
                                ┌─────────────────────────┐
                                │ Tool: Regras de negócio │
                                └─────────────────────────┘
                                              │
                                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   AI Gateway Corporativo (.NET)                    │
│ policy engine | PII sanitization | guardrails | routing | audit    │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│              LiteLLM / adaptador multi-provider / LLM              │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                OpenAI / Anthropic / OCI / modelos internos         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔀 6. Por que usar um serviço de agentes separado do Gateway

O **Agent Service** deve conter:
- lógica multi-etapa;
- uso de tools;
- estado;
- raciocínio operacional;
- handoff entre papéis especializados, se necessário.

O **AI Gateway** deve conter:
- políticas;
- anonimização/pseudonimização;
- guardrails;
- roteamento entre modelos;
- observabilidade;
- custos e auditoria.

> [!tip] Separação de responsabilidades
> Essa separação reduz acoplamento, preserva governança e impede que o gateway vire um orquestrador de negócio.

---

## 💻 7. Linguagem e framework recomendados

### 🏆 7.1. Recomendação principal
Para o primeiro agente, a recomendação é:

- **Agent Service**: Python
- **AI Gateway**: .NET
- **Persistência operacional / auditoria**: Oracle e stores específicos
- **LLM access**: sempre via Gateway

**Justificativa**

Para agentes, hoje Python continua sendo a opção mais produtiva para prototipar e evoluir padrões agentic com tools, estado, workflow e human-in-the-loop. O framework mais aderente a um cenário corporativo controlado é **LangGraph**, especialmente quando se quer modelar o fluxo como grafo e manter checkpoints, persistência e retomada. Como alternativa mais leve e direta para tool calling, tracing e handoffs, o **OpenAI Agents SDK** também é uma boa escolha.

### 🔄 7.2. Alternativas
- **LangGraph**: melhor quando você quer workflow explícito, checkpoints, state graph e maior controle do fluxo.
- **OpenAI Agents SDK**: melhor quando você quer um runtime enxuto, com primitives simples, tool calling, handoffs e tracing.
- **Microsoft Agent Framework**: alternativa forte para padronização futura em .NET e Python, principalmente se a organização quiser convergir mais tarde para um stack Microsoft de agentes.

---

## 🤖 8. Forma do agente: workflow-agent híbrido

Para esse caso de uso, a melhor forma não é um agente totalmente livre. O desenho recomendado é um **workflow-agent híbrido**:

### ⚙️ Partes determinísticas
- identificar o lote / pagamento / período;
- consultar bases obrigatórias;
- montar evidências;
- aplicar critérios mínimos de confiança;
- decidir se é necessário escalar.

### 🧠 Partes agentic
- decidir a ordem ideal de algumas consultas;
- pedir ferramentas adicionais se necessário;
- sintetizar causa provável;
- gerar resposta natural e resumo executivo.

### ✅ Vantagens
- reduz risco de alucinação;
- facilita teste;
- melhora auditoria;
- limita autonomia em áreas críticas.

---

## 🧩 9. Componentes do Agent Service

### 9.1. Agent API / Facade
Responsável por:
- receber pergunta do usuário;
- iniciar ou continuar sessão;
- encaminhar para o orchestrator;
- devolver resposta em streaming ou completa.

### 9.2. Session State Store
Armazena:
- thread da conversa;
- estado do caso analisado;
- checkpoints do fluxo;
- histórico resumido das evidências;
- decisões intermediárias.

### 9.3. Orchestrator / Planner
Coordena:
- interpretação da intenção;
- seleção das tools;
- ordem de execução;
- chamadas ao Gateway;
- validação do resultado final.

### 9.4. Tool Registry
Catálogo interno de tools permitidas:
- tool name;
- schema de entrada;
- schema de saída;
- sistema de destino;
- classificação de segurança;
- necessidade de aprovação.

### 9.5. Confidence & Escalation Service
Decide:
- se a resposta é suficientemente confiável;
- se faltam evidências;
- se o caso deve ser escalado para humano;
- se o agente pode apenas sugerir hipótese ou recomendar ação.

### 9.6. Audit Adapter
Envia para a trilha corporativa:
- sequência de tool calls;
- IDs consultados;
- tempos de execução;
- prompts sanitizados;
- decisão final do agente.

---

## 🔧 10. Tools do agente

> [!info] Padrão de tools
> As tools devem ser APIs ou wrappers internos, tipados, auditáveis e com schemas de entrada/saída bem definidos.

### 10.1. ConsultarPagamentos
**Objetivo:** obter pagamentos por lote, data, status, favorecido ou identificador.

**Entrada exemplo:**
```json
{
  "loteId": "8457",
  "data": "2026-03-27"
}
```

**Saída exemplo:**
```json
{
  "loteId": "8457",
  "quantidadePagamentos": 132,
  "totalEsperado": 187450.22,
  "totalProcessado": 179320.22,
  "divergencia": 8130.00,
  "status": "DIVERGENTE"
}
```

### 10.2. ObterDetalhesPagamento
Retorna detalhes de um pagamento específico:
- valor;
- favorecido;
- convênio;
- conta;
- timestamps;
- códigos de erro;
- status de envio e retorno.

### 10.3. ConsultarTitulosOuBoletos
Busca vínculo do pagamento com título, boleto ou obrigação financeira.

### 10.4. ConsultarEventosOperacionais
Busca:
- logs de integração;
- códigos de rejeição;
- timeouts;
- retries;
- eventos de fila;
- falhas de autenticação ou conectividade.

### 10.5. ConsultarRegrasDeNegocio
Retorna regras aplicáveis:
- tolerância de valor;
- janelas de processamento;
- obrigatoriedade de campos;
- regras por tipo de pagamento ou concessionária.

### 10.6. ConsultarHistoricoDeDivergenciasSemelhantes
Retorna casos anteriores semelhantes para enriquecer hipótese e priorização.

### 10.7. CriarTicketOuEncaminhamento
Ferramenta opcional e protegida para abertura de tratativa, sempre sujeita a confirmação humana.

---

## 👥 11. Papéis especializados (opcional)

Em vez de um único agente genérico, é possível estruturar papéis especializados, ainda dentro do mesmo serviço:

### 11.1. Intake Agent
- interpreta a pergunta;
- identifica entidade principal;
- normaliza parâmetros.

### 11.2. Investigation Agent
- chama tools;
- coleta evidências;
- monta hipótese.

### 11.3. Explanation Agent
- escreve resumo executivo e técnico;
- organiza evidências;
- produz resposta para o usuário.

### 11.4. Escalation Agent
- decide se encaminha para humano;
- propõe ticket, fila ou tratativa.

> [!tip] Recomendação para o piloto
> Começar com **um único agente orquestrado + tools**, e só depois avaliar multi-agent.

---

## 🔄 12. Fluxo ponta a ponta

### 12.1. Exemplo de pergunta
> "Por que o pagamento do lote 8457 ficou divergente ontem?"

### 12.2. Etapas

#### Etapa 1 — Intake
O agente extrai:
- tipo da pergunta = investigação de divergência;
- entidade = lote 8457;
- janela temporal = ontem.

#### Etapa 2 — Planejamento inicial
O agente decide um plano mínimo:
1. consultar resumo do lote;
2. consultar detalhes dos pagamentos divergentes;
3. consultar eventos operacionais;
4. consultar regras aplicáveis;
5. sintetizar hipótese.

#### Etapa 3 — Tool calls
O agente chama:
- `ConsultarPagamentos`
- `ObterDetalhesPagamento`
- `ConsultarEventosOperacionais`
- `ConsultarRegrasDeNegocio`

#### Etapa 4 — Montagem do contexto mínimo
O agente transforma os resultados brutos em contexto enxuto:
- remove campos desnecessários;
- agrupa eventos relevantes;
- estrutura evidências.

#### Etapa 5 — Chamada ao AI Gateway
O agente envia ao Gateway:
- `taskType = divergence_analysis`
- `businessDomain = payments`
- `dataClassification = restricted`
- `requiresStructuredOutput = true`
- prompt + evidências resumidas

#### Etapa 6 — Governança no Gateway
O Gateway:
- autentica o serviço chamador;
- aplica políticas;
- detecta PII;
- mascara ou pseudonimiza;
- aplica guardrails;
- escolhe modelo;
- audita.

#### Etapa 7 — Resposta estruturada do LLM
Exemplo lógico:
```json
{
  "provavelCausa": "falha de integração no retorno bancário",
  "confianca": 0.82,
  "evidencias": [
    "Timeout no adaptador bancário às 18:42",
    "35 pagamentos ficaram sem confirmação de retorno",
    "Regra exige retorno síncrono para fechamento do lote"
  ],
  "acaoRecomendada": "reprocessar conciliação e abrir incidente para integração bancária",
  "requerRevisaoHumana": true
}
```

#### Etapa 8 — Validação final
O agente:
- valida esquema;
- verifica confiança mínima;
- decide se precisa pedir mais uma tool;
- ou apresenta o resultado.

#### Etapa 9 — Resposta ao usuário
O usuário recebe:
- resumo executivo;
- causa provável;
- evidências;
- ação sugerida;
- indicação de revisão humana.

---

## 📐 13. Contrato entre Agent Service e AI Gateway

### 13.1. Requisição
```json
{
  "correlationId": "e2b2d7c8-42f7-4d6a-9f2a-61bb1f7dc410",
  "sourceSystem": "payments-agent-service",
  "businessDomain": "payments",
  "taskType": "divergence_analysis",
  "dataClassification": "restricted",
  "requiresStructuredOutput": true,
  "expectedSchema": "DivergenceAnalysisResultV1",
  "prompt": "Analise as evidências abaixo e classifique a causa provável da divergência.",
  "contextItems": [
    {
      "type": "lot_summary",
      "content": "Lote 8457: total esperado 187450.22, total processado 179320.22, divergência 8130.00."
    },
    {
      "type": "events",
      "content": "Timeout no adaptador bancário às 18:42; 35 pagamentos sem retorno confirmado."
    },
    {
      "type": "rules",
      "content": "Fechamento exige retorno confirmado para composição do lote."
    }
  ]
}
```

### 13.2. Resposta
```json
{
  "correlationId": "e2b2d7c8-42f7-4d6a-9f2a-61bb1f7dc410",
  "model": "provider/model",
  "policyVersion": "payments-divergence-v3",
  "content": {
    "provavelCausa": "falha de integração no retorno bancário",
    "confianca": 0.82,
    "evidencias": [
      "Timeout no adaptador bancário às 18:42",
      "35 pagamentos ficaram sem confirmação de retorno"
    ],
    "acaoRecomendada": "reprocessar conciliação e abrir incidente",
    "requerRevisaoHumana": true
  },
  "usage": {
    "inputTokens": 1480,
    "outputTokens": 210
  }
}
```

---

## 💬 14. Prompting strategy

### 14.1. Instrução do agente
O agente deve operar com instruções restritas:
- usar tools para coletar fatos;
- não inventar evidências;
- declarar incerteza;
- preferir hipóteses verificáveis;
- emitir saída estruturada;
- propor revisão humana quando confiança for insuficiente.

### 14.2. Instrução do Gateway
O Gateway pode reforçar:
- "não tente reconstruir PII";
- "não extrapole além das evidências";
- "retorne apenas no schema esperado";
- "se os dados forem insuficientes, indique insuficiência".

---

## 🛡️ 15. Guardrails e governança

### 15.1. No Agent Service
- whitelisting de tools;
- schemas estritos;
- timeouts;
- limitação de profundidade de iteração;
- limitação de número de tool calls;
- confidence threshold;
- bloqueio de ação automática em sistemas críticos.

### 15.2. No AI Gateway
- autenticação do serviço;
- decisão de política;
- detecção de PII;
- pseudonimização;
- output validation;
- auditoria;
- routing entre modelos.

### 15.3. Human-in-the-loop
Deve ser obrigatório quando:
- confiança abaixo do limite;
- sugestão envolver ação operacional com impacto financeiro;
- houver inconsistência entre tools;
- o caso envolver cliente específico e dado sensível;
- houver tentativa de criação de ticket com dados críticos.

---

## 💾 16. Estado e memória

### 16.1. Memória de execução
Guardar:
- pergunta original;
- plano corrente;
- tools já chamadas;
- evidências relevantes;
- hipótese corrente;
- justificativa da decisão.

### 16.2. Memória de longo prazo

> [!warning] Atenção
> Não deve armazenar indiscriminadamente conteúdo sensível. O ideal é:
> - armazenar resumos;
> - armazenar IDs de referência;
> - armazenar embeddings apenas quando houver política e base legal;
> - nunca usar memória livre como substituto de fonte oficial de negócio.

---

## 🗺️ 17. Estratégia de implementação por fases

### Fase 1 — Piloto controlado
- 1 agente;
- 4 tools de leitura;
- sem escrita em sistemas críticos;
- 1 tipo de caso: divergência de pagamentos;
- resposta estruturada;
- human review para fechamento.

### Fase 2 — Produção assistida
- ampliar cobertura de causas;
- adicionar histórico de casos similares;
- integrar abertura de ticket com aprovação;
- dashboards operacionais.

### Fase 3 — Expansão
- novos agentes por domínio;
- handoff entre papéis;
- mais automação controlada;
- avaliação contínua de qualidade.

---

## 🔩 18. Escolha de framework

### 🏆 18.1. Opção recomendada para o piloto: LangGraph
**Motivos**
- ótimo encaixe para workflow-agent híbrido;
- estado explícito;
- checkpoints;
- retomada após falha;
- human-in-the-loop;
- boa adequação a fluxos corporativos auditáveis.

**Desenho de grafo sugerido**
```text
Start
  ↓
ParseIntent
  ↓
PlanMinimumSteps
  ↓
CallTool_Pagamentos
  ↓
NeedMoreEvidence?
 ├─ sim → CallTool_Events / CallTool_Rules / CallTool_History
 └─ não
  ↓
BuildContextForGateway
  ↓
CallAIGateway
  ↓
ValidateStructuredOutput
  ↓
ConfidenceCheck
 ├─ baixa → EscalateHuman
 └─ adequada → RespondUser
```

### 🔄 18.2. Opção alternativa: OpenAI Agents SDK
**Motivos**
- runtime simples;
- tools, handoffs e tracing integrados;
- boa produtividade quando o fluxo não precisa ser modelado em grafo de forma tão explícita.

### 🏢 18.3. Opção corporativa futura: Microsoft Agent Framework
**Motivos**
- converge .NET e Python;
- oferece caminho de evolução mais alinhado ao ecossistema Microsoft;
- interessante para padronização futura, especialmente se a organização quiser reduzir a distância entre agentes e serviços .NET.

---

## 🚨 19. Riscos principais e mitigação

### Risco 1 — Alucinação causal
**Mitigação:** tool-first, structured output, confidence threshold, revisão humana.

### Risco 2 — Exposição de dados sensíveis
**Mitigação:** Gateway obrigatório, minimização de contexto, PII sanitization.

### Risco 3 — Agente agir além do permitido
**Mitigação:** tools com escopo restrito, approvals, whitelisting, sem acesso direto a sistemas críticos.

### Risco 4 — Fluxo difícil de operar
**Mitigação:** workflow híbrido, checkpoints, tracing, métricas.

### Risco 5 — Custo alto
**Mitigação:** contexto mínimo, caching de consultas, routing por perfil de modelo, limites por sessão.

---

## 🏁 20. Recomendação final

Para o cenário descrito, a melhor solução inicial é:

```text
Portal / API interna
   ↓
Agent Service (Python, workflow-agent híbrido)
   ├─ tools corporativas de leitura
   ├─ confidence service
   ├─ escalation
   └─ session state
   ↓
AI Gateway corporativo (.NET)
   ├─ policy engine
   ├─ PII sanitization
   ├─ guardrails
   ├─ routing
   └─ auditoria
   ↓
LiteLLM / provider adapter
   ↓
LLMs externos ou internos
```

### Decisão prática
- usar **Python** para o primeiro agente;
- usar **.NET** para o AI Gateway;
- começar com **workflow-agent híbrido**;
- limitar o agente a **análise e recomendação**, não execução autônoma;
- exigir **human-in-the-loop** em ações que possam impactar operação financeira.

---

## 📝 21. Próximos passos sugeridos

1. Definir o contrato HTTP entre Agent Service e AI Gateway.
2. Catalogar as primeiras 4 tools de leitura.
3. Definir o schema `DivergenceAnalysisResultV1`.
4. Implementar piloto com histórico de auditoria completo.
5. Rodar avaliação com casos reais anonimizados.
6. Só depois considerar multi-agent ou automação de ações.

---

## 📚 22. Referências de arquitetura e frameworks

Este desenho se apoia em capacidades atualmente documentadas pelos frameworks mais relevantes:
- LangGraph para orquestração com grafo, persistência e checkpoints;
- LangChain Agents construídos sobre LangGraph;
- OpenAI Agents SDK para tools, handoffs e tracing;
- Microsoft Agent Framework como evolução recente para agentes em .NET e Python;
- AI Gateway corporativo como camada separada de governança.
