---
title: "Claude Code - Anonimizacao para Intelligence Portal v2"
date: 2026-04-13
tags:
  - pesquisa
  - ai
  - arquitetura
  - seguranca
  - status/revisado
area: arquitetura
---

# Claude Code - Anonimizacao para Intelligence Portal v2

## Contexto

Documento consolidado sobre estrategia de anonimização, tokenização e roteamento de LLMs externos para um portal financeiro.

## Resumo

Esta nota consolida uma proposta arquitetural para uso de LLMs externos com dados financeiros, combinando tokenização reversivel, fator proporcional para valores numericos, gateway central de IA e separação entre computacao deterministica local e interpretacao remota por LLM.

## Conteudo consolidado

# Intelligence Portal v2 — Estratégia de Anonimização para Uso de LLMs Externos

## 1. Contexto e motivação

O Intelligence Portal v2 fará uso intensivo de modelos LLM. O provedor Oracle OCI Generative AI, apesar de oferecer a vantagem de executar modelos em datacenters no Brasil (dados estáticos e sob controle regional), apresenta desempenho e qualidade inferior aos modelos disponíveis pela OpenAI, Anthropic e Google.

Para utilizar provedores externos de maior qualidade, é necessário um plano de anonimização dos dados do sistema — que são de natureza financeira — antes de serem enviados a esses modelos.

Este documento consolida as decisões arquiteturais, padrões de modelagem e estratégias de anonimização discutidos para o novo sistema.

---

## 2. Estratégias de anonimização

### 2.1 Tokenização reversível

Abordagem principal e mais indicada para o cenário do portal. Consiste em substituir valores sensíveis por tokens opacos antes do envio ao LLM, e reverter os tokens na resposta.

**Exemplo:**

|Dado original|Token|
|---|---|
|Petrobras|`[EMPRESA_001]`|
|33.000.167/0001-01|`[CNPJ_001]`|
|R$ 502.000.000,00|`[VALOR_001]`|

O LLM trabalha com os tokens e consegue raciocinar sobre relações ("a empresa `[EMPRESA_001]` teve `[VALOR_001]` de receita") sem ter acesso aos dados reais.

### 2.2 Generalização / perturbação (fator proporcional)

Útil quando o LLM precisa operar sobre valores numéricos para análise de tendências, variações percentuais e rankings. Todos os valores de uma sessão são multiplicados por um fator aleatório fixo (ex: 0.73x).

**Características:**

- Proporções, percentuais e rankings permanecem corretos.
- Valores absolutos não são reais, eliminando risco de cruzamento.
- A reversão exige "desperturbar" valores calculados pelo LLM na resposta.

**Limitação:** falha em cenários que dependem de valor absoluto (ex: "a empresa está acima do threshold de R$ 100M?").

### 2.3 Dados sintéticos para few-shot / prompts de sistema

Nos prompts que definem o comportamento do modelo (system prompts, exemplos few-shot), nunca utilizar dados reais. Gerar dados fictícios com a mesma estrutura.

---

## 3. Impacto na modelagem orientada a objetos

### 3.1 Campo sensível como metadado

Cada entidade do domínio (Cliente, Contrato, Transação) deve declarar quais campos são sensíveis. Isso pode ser implementado via:

- Anotações / decorators nas classes.
- Registro central de políticas.
- Interface como `ISensitiveDataHolder`.

O sistema precisa saber, em tempo de execução, o que anonimizar — de forma declarativa, não heurística.

### 3.2 AnonymizationContext

Serviço que gerencia o mapeamento bidirecional `token ↔ valor real` durante uma sessão de interação com o LLM.

**Responsabilidades:**

- Criar tokens opacos para cada valor sensível.
- Armazenar o mapa de correspondência.
- Executar a reversão (detokenização) na resposta.
- Escopo efêmero (vive apenas durante a chamada) ou brevemente persistido para auditoria.

### 3.3 LLM Gateway / Proxy

Componente centralizado que intercepta todas as chamadas a provedores externos. Nenhuma parte do sistema se comunica diretamente com OpenAI, Anthropic ou Google.

**Responsabilidades:**

- Aplicar anonimização na ida e de-anonimização na volta.
- Roteamento entre provedores (OCI vs. externos).
- Retry, rate limiting, logging e auditoria.

---

## 4. Impacto no banco de dados

### 4.1 Tabela de políticas de sensibilidade

Mapeia `entidade → campo → nível de sensibilidade → estratégia de anonimização`.

Permite que regras de anonimização mudem sem recompilação do sistema. A estratégia é vinculada ao **tipo de prompt/tarefa**, não apenas ao campo — o mesmo campo `receita` pode ter tratamentos diferentes dependendo do que o LLM vai fazer com ele.

### 4.2 Log de auditoria de chamadas a LLMs externos

Registra:

- O prompt enviado (já anonimizado).
- A resposta recebida.
- Timestamp e provedor utilizado.
- Versão do modelo e do prompt template.

Essencial para compliance com LGPD e regulação financeira.

### 4.3 Cache de respostas

Se respostas do LLM forem armazenadas em cache, devem estar na forma de-anonimizada e marcadas com a versão do modelo e do prompt, pois esses resultados envelhecem.

---

## 5. Arquitetura geral — duas rotas de dados

O sistema opera com duas rotas distintas:

```
┌─────────────────────────────────────────────────────────┐
│                      Aplicação                          │
│                  (frontend + backend)                   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    LLM Gateway                          │
│              (roteamento + orquestração)                │
└────────┬────────────────────────────────┬───────────────┘
         │                                │
         │ Dados reais                    │ Dados reais
         ▼                                ▼
┌──────────────────┐         ┌────────────────────────┐
│  Oracle OCI      │         │  Proxy de anonimização │
│  GenAI           │         │  (tokenizar /          │
│  (datacenter BR) │         │   detokenizar)         │
│  Acesso direto   │         └───────────┬────────────┘
└──────────────────┘                     │ Dados tokenizados
                                         ▼
                            ╭╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╮
                            ┆  Provedores externos     ┆
                            ┆  OpenAI, Anthropic,      ┆
                            ┆  Google                  ┆
                            ┆  (fora do datacenter BR) ┆
                            ╰╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╯
```

**Rota esquerda (Oracle OCI):** dados vão diretos ao datacenter brasileiro — sem necessidade de anonimização. O gateway mantém a mesma interface, apenas com a flag de anonimização desligada.

**Rota direita (provedores externos):** dados passam pelo proxy de anonimização antes de sair do ambiente controlado.

---

## 6. Pipeline de anonimização — fluxo detalhado

```
 CONTEXTO ORIGINAL (dados reais)
 "A empresa Petrobras, CNPJ 33.000.167/0001-01,
  teve receita de R$ 502.000.000,00 no Q3"
                    │
                    ▼
 SCANNER DE PII (políticas declarativas)
 Detecta: nome empresa, CNPJ, valor monetário,
 conta, CPF...
                    │
                    ▼
 TOKENIZADOR (substitui por tokens opacos)
 "A empresa [EMPRESA_001], CNPJ [CNPJ_001],
  teve receita de [VALOR_001] no Q3"

  ┌─────────────────────────────────────────┐
  │  Mapa token ↔ valor (efêmero)          │
  │  [EMPRESA_001] = Petrobras             │
  │  [CNPJ_001] = 33.000.167/0001-01      │
  │  [VALOR_001] = R$ 502.000.000,00      │
  └────────────────────┬────────────────────┘
                       │
                    ▼  │
 CHAMADA AO LLM EXTERNO                    │
 (prompt com tokens opacos)                 │
 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─                  │
 Fora do seu ambiente                       │
 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─                  │
                    │                       │
                    ▼                       │
 DETOKENIZADOR ◄────────────────────────────┘
 (reverte tokens → valores reais)
                    │
                    ▼
 RESPOSTA FINAL (dados reais restaurados)
 "A Petrobras apresentou receita
  de R$ 502.000.000,00..."
```

---

## 7. Análise de risco: valores monetários

### 7.1 Valores isolados não são PII

Um número como R$ 502.000.000,00, sozinho, não identifica ninguém. O risco está na **combinação de valor + contexto** que permite **re-identificação por quasi-identifiers**: nenhum campo sozinho identifica, mas a combinação de dois ou três campos sim.

**Exemplo de risco:** se o prompt diz `[EMPRESA_001] teve receita de R$ 502.000.000,00 no Q3 2025`, um agente com acesso a esses dados poderia cruzar o valor com dados públicos e deduzir a identidade da empresa.

### 7.2 Estratégia híbrida por tipo de operação

|Tipo de operação|Estratégia|Justificativa|
|---|---|---|
|Comparação com thresholds, classificação por faixa|Valores reais (arredondados)|R$ 502.347.891,23 → R$ 502.000.000. Reduz unicidade mantendo funcionalidade|
|Variação percentual, ranking, tendência|Fator proporcional|Proporções idênticas, valores absolutos irreais|
|Relatórios narrativos (sem cálculo)|Tokenização completa|LLM usa `[VALOR_001]`, detokenização resolve|

### 7.3 Política vinculada ao tipo de tarefa

A política de anonimização não se vincula apenas ao campo, mas ao **contexto de uso**. O mesmo campo `receita` pode ter três tratamentos diferentes dependendo da tarefa que o LLM vai executar.

---

## 8. Caso de uso: apuração de saldo caixa multi-período

### 8.1 Prompt do usuário

O usuário solicita no chat:

> Apure o saldo caixa hoje detalhado por empresa, o saldo caixa de 180 dias atrás, calcule a variação absoluta e percentual, faça um comentário sobre a evolução e gere KPIs coloridos.

Formato de saída esperado:

|empresa|saldo caixa hoje|saldo caixa 180d|variação|variação %|comentário|kpi|
|---|---|---|---|---|---|---|

### 8.2 Análise do fluxo

|Item|Natureza|Quem executa|
|---|---|---|
|Saldo caixa hoje|Determinístico (query SQL)|Backend (via tool call)|
|Saldo caixa 180d|Determinístico (query SQL)|Backend (via tool call)|
|Variação absoluta|Determinístico (aritmética)|Backend ou LLM|
|Variação percentual|Determinístico (aritmética)|Backend ou LLM|
|Comentário qualitativo|Interpretativo (linguagem natural)|LLM|
|KPI (verde/amarelo/vermelho)|Classificação qualitativa|LLM ou backend (regras fixas)|

### 8.3 Padrão "compute locally, interpret remotely"

- **Backend sempre busca os dados brutos** via tool call (query no banco).
- **Aritmética fixa e previsível** (diferença entre 2 datas) pode ser computada localmente.
- **LLM recebe apenas o mínimo necessário** para gerar comentários e KPIs.

**Dados que saem para o LLM externo (exemplo com proteção):**

|empresa|var %|saldo_faixa|
|---|---|---|
|[EMP_001]|+12.4%|alto|
|[EMP_002]|-3.1%|médio|
|[EMP_003]|-18.7%|baixo|

O backend monta a tabela final substituindo tokens e preenchendo colunas de valores reais.

---

## 9. Cenário de N datas: complexidade do "compute locally"

### 9.1 O problema da explosão combinatória

Com duas datas, a aritmética é trivial. Com N datas (3, 4, 5 períodos), o que o usuário pede não é mais uma subtração — é uma **análise combinatória** entre períodos:

- Variação entre cada par consecutivo.
- Variação acumulada do primeiro ao último.
- Qual período teve maior crescimento.
- Qual empresa teve comportamento mais volátil.
- Tendência geral (crescente, decrescente, oscilante).

Com 5 datas, existem 10 pares possíveis e múltiplos ângulos de análise. Implementar isso no backend significa construir um mini-engine de análise comparativa com regras para cada cenário — frágil e difícil de manter.

### 9.2 Critério de decisão

|Natureza da lógica|Onde executar|Exemplo|
|---|---|---|
|Fixa e previsível|Backend (compute locally)|Buscar saldo de N datas, formatar tabela|
|Variável e interpretativa|LLM (interpret remotely)|Decidir quais comparações são relevantes entre N cenários, gerar comentários|

### 9.3 Solução recomendada

A tool permanece simples: recebe N datas, retorna N colunas de saldo por empresa. Os dados vão ao LLM com **fator proporcional aplicado** — as variações percentuais, rankings e tendências se preservam. O LLM decide quais comparações e comentários são relevantes (é exatamente o que ele faz bem).

O backend faz a montagem final da tabela, revertendo os valores perturbados e substituindo os tokens de identificação.

---

## 10. Resumo das decisões arquiteturais

|Decisão|Escolha|Justificativa|
|---|---|---|
|Abordagem de anonimização|Tokenização reversível + fator proporcional (híbrido)|Cobre tanto cenários narrativos quanto aritméticos|
|Detecção de dados sensíveis|Declarativa (metadados no modelo)|Mais robusta que regex/NER; NER pode ser camada extra|
|Roteamento de LLMs|Gateway centralizado|Uniformidade, troca de provedor sem mudar lógica de negócio|
|Divisão de responsabilidades|Compute locally, interpret remotely|Reduz superfície de exposição, custo de tokens e latência|
|Política de anonimização|Vinculada ao tipo de tarefa, não ao campo|Mesmo campo pode ter tratamentos diferentes por contexto|
|Aritmética com N cenários|Delegada ao LLM com dados protegidos|Evita engine de regras frágil no backend|
|Oracle OCI|Mesma interface do gateway, sem anonimização|Permite trocar provedor transparentemente|
|Auditoria|Log de todas as chamadas externas|Compliance LGPD e regulação financeira|

---

## 11. Próximos passos sugeridos

1. **Modelagem OO detalhada** — definir interfaces, classes e decorators para campos sensíveis e políticas de anonimização.
2. **Schema do banco de dados** — tabelas de políticas, auditoria e cache de respostas.
3. **Implementação do AnonymizationContext** — serviço de tokenização/detokenização com suporte a fator proporcional.
4. **Prototipação do LLM Gateway** — roteamento Oracle OCI vs. externos, com pipeline de anonimização plugável.
5. **Definição de prompts e tools** — templates para os cenários de uso identificados (saldo caixa, análise multi-período, etc.).
6. **Testes de re-identificação** — validar que dados anonimizados não permitem cruzamento com fontes públicas.

## Notas relacionadas

- [[anonimizacao-de-dados-para-llms-em-sistemas-financeiros]]
- [[anonimizacao-e-contexto-minimo-para-llms]]
- [[ai-gateway-arquitetura-financeiro]]
