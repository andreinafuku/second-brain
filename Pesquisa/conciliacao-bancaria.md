---
title: "Conciliação Bancária — Conceito, Importância e Automação com Open Finance"
tags:
  - pesquisa
  - arquitetura/open-finance
  - glossario
  - status/consolidado
date: 2026-04-13
area: tesouraria
---

# 🏦 Conciliação Bancária

## 📌 O que é conciliação bancária?

Conciliação bancária é o **processo de comparação e verificação** entre os registros financeiros internos de uma empresa (lançamentos contábeis, contas a pagar, contas a receber, movimentações no ERP) e os **extratos fornecidos pelas instituições financeiras** (bancos, fintechs, instituições de pagamento).

O objetivo é garantir que **toda movimentação financeira registrada internamente tenha uma correspondência exata no extrato bancário**, e vice-versa. Quando há divergência, ela precisa ser identificada, investigada e corrigida.

> [!abstract] Definição resumida
> Conciliação bancária é o ato de **cruzar os registros internos da empresa com os registros do banco**, garantindo que ambos reflitam a mesma realidade financeira.

### 🔍 O que se compara?

```text
┌─────────────────────────────┐     ┌─────────────────────────────┐
│   Registros internos        │     │   Extrato bancário          │
│   (ERP / Contabilidade)     │     │   (Banco / Instituição)     │
│                             │     │                             │
│  - Pagamentos emitidos      │ ←→  │  - Débitos efetivados       │
│  - Recebimentos esperados   │ ←→  │  - Créditos identificados   │
│  - Transferências entre     │ ←→  │  - Transferências           │
│    contas                   │     │    processadas              │
│  - Tarifas previstas        │ ←→  │  - Tarifas cobradas         │
│  - Aplicações / resgates    │ ←→  │  - Movimentações de         │
│                             │     │    investimento              │
└─────────────────────────────┘     └─────────────────────────────┘
                    │                           │
                    └───────────┬───────────────┘
                                ▼
                    ┌───────────────────────┐
                    │   CONCILIAÇÃO         │
                    │   Matching            │
                    │   Identificação de    │
                    │   divergências        │
                    │   Resolução           │
                    └───────────────────────┘
```

---

## 🎯 Para que serve?

A conciliação bancária serve a múltiplos propósitos que vão além de simplesmente "conferir se os números batem":

### 1. Integridade contábil

Garante que o **saldo contábil** da empresa reflita fielmente o **saldo real** disponível no banco. Sem conciliação, a contabilidade pode apresentar uma realidade distorcida — a empresa pode acreditar que tem mais (ou menos) dinheiro do que realmente possui.

### 2. Detecção de erros e fraudes

- **Pagamentos duplicados**: fornecedor cobrado duas vezes
- **Créditos não identificados**: recebimentos que chegaram ao banco mas não foram registrados internamente
- **Débitos não autorizados**: cobranças indevidas, tarifas incorretas, fraudes bancárias
- **Erros de digitação**: valores lançados incorretamente no ERP
- **Diferenças de data**: pagamentos processados em datas diferentes do previsto (float bancário)

### 3. Gestão de fluxo de caixa

A conciliação atualiza a **posição de caixa real** da empresa. Sem ela, decisões de investimento, pagamento e captação de recursos são tomadas com base em dados desatualizados ou incorretos.

### 4. Compliance e auditoria

- **Auditoria interna e externa**: auditores exigem evidências de conciliação regular
- **Controles internos (SOX, COSO)**: empresas de capital aberto e grandes corporações devem manter controles de conciliação como parte de seus frameworks de controle interno
- **Fiscalização tributária**: divergências entre registros e extratos podem gerar questionamentos da Receita Federal

### 5. Fechamento contábil

A conciliação é **pré-requisito para o fechamento mensal**. Sem ela concluída, o balanço patrimonial e a DRE não podem ser considerados confiáveis.

---

## 🏢 Por que as empresas precisam desse processo?

### A realidade operacional

Na prática, os registros internos e os extratos bancários **quase nunca coincidem perfeitamente**, mesmo quando tudo está correto. As divergências são naturais e surgem por diversos motivos:

#### Divergências temporais (timing differences)

| Situação | Registro interno | Extrato bancário |
|---|---|---|
| Cheque emitido | Registrado na emissão | Debitado quando compensado (dias depois) |
| Boleto recebido | Registrado na emissão da cobrança | Creditado após compensação (D+1 ou D+2) |
| TED enviada | Registrada na aprovação | Debitada no processamento (mesmo dia ou D+1) |
| Tarifa bancária | Não prevista no ERP | Debitada automaticamente pelo banco |
| Rendimento de aplicação | Estimado pelo ERP | Creditado com valor real (pode diferir) |

#### Divergências de valor

- **IOF** sobre operações de crédito não previsto no lançamento original
- **Taxas bancárias** variáveis ou reajustadas
- **Juros e multas** em pagamentos atrasados
- **Descontos** concedidos ou obtidos não registrados

#### Divergências operacionais

- **Pagamentos pendentes**: aprovados internamente mas ainda não processados pelo banco
- **Depósitos em trânsito**: valores recebidos pelo banco mas ainda não identificados
- **Estornos**: devoluções processadas pelo banco sem contrapartida no ERP
- **Débitos automáticos**: cobranças de concessionárias, seguros, aluguéis

### Consequências de não conciliar

> [!danger] Riscos de não executar conciliação bancária
> - **Decisões financeiras baseadas em dados errados** — ex: investir um valor que na verdade já foi debitado
> - **Pagamentos duplicados não detectados** — perda financeira direta
> - **Fraudes não identificadas** — desvios podem passar meses sem detecção
> - **Multas e juros** por pagamentos que falharam sem que ninguém percebesse
> - **Parecer de auditoria com ressalvas** — impacto na credibilidade da empresa
> - **Demonstrações financeiras incorretas** — risco regulatório e reputacional

### O desafio escala com o porte da empresa

| Porte | Volume típico | Bancos | Complexidade |
|---|---|---|---|
| **Pequena empresa** | Dezenas a centenas de transações/mês | 1-2 bancos | Baixa, mas crítica — erros impactam o caixa rapidamente |
| **Média empresa** | Centenas a milhares de transações/mês | 2-5 bancos | Moderada — já requer processo estruturado |
| **Grande empresa** | Milhares a dezenas de milhares de transações/dia | 5-20+ bancos | Alta — múltiplas contas, moedas, filiais, centros de custo |
| **Grupo econômico** | Centenas de milhares de transações/dia | 20+ bancos, múltiplas entidades jurídicas | Muito alta — consolidação intercompany, eliminações |

---

## ⚙️ Como a conciliação é feita tradicionalmente

### Processo manual (ainda comum em PMEs)

```text
1. Funcionário acessa internet banking de cada banco
2. Exporta extrato (OFX, CSV, PDF)
3. Importa no ERP ou abre em planilha
4. Compara linha a linha com os lançamentos internos
5. Marca os itens conciliados (matching)
6. Investiga e trata os itens pendentes
7. Registra ajustes contábeis quando necessário
8. Gera relatório de conciliação para auditoria
```

#### Problemas do processo manual

- **Lento**: uma empresa com 5 bancos e milhares de transações pode levar **dias** para conciliar um mês
- **Propenso a erros humanos**: matching visual é falho, especialmente com volumes altos
- **Dependente de arquivos bancários**: formatos variam por banco (CNAB 240, CNAB 400, OFX, CSV proprietário)
- **Sem tempo real**: a conciliação é feita dias ou semanas após as transações, reduzindo sua utilidade para gestão de caixa
- **Não escalável**: dobrar o volume de transações mais que dobra o esforço de conciliação

### Processo semi-automatizado (comum em grandes empresas)

```text
1. Integração via arquivo CNAB ou API proprietária com cada banco
2. Importação automática de extratos no ERP
3. Regras de matching automático (valor + data + referência)
4. Fila de exceções para tratamento manual
5. Workflow de aprovação para ajustes
6. Relatório automatizado
```

#### Limitações da semi-automação

- **Integrações ponto a ponto**: cada banco exige uma integração específica (CNAB 240 do Itaú ≠ CNAB 240 do Bradesco)
- **Manutenção custosa**: mudanças de layout bancário quebram integrações
- **Conectividade frágil**: muitas integrações dependem de VAN (Value-Added Network), SFTP ou troca de arquivos — não são real-time
- **Cobertura parcial**: nem todos os bancos oferecem APIs ou formatos compatíveis
- **Alto custo de setup**: integrar cada novo banco ao ERP pode levar semanas ou meses

---

## 🌐 Como o Open Finance transforma a conciliação bancária

O [[open-finance-brasil|Open Finance]] muda fundamentalmente a maneira como os dados bancários chegam até a empresa. Em vez de depender de integrações proprietárias com cada banco, a empresa (ou seu software de tesouraria) pode acessar os dados de **todas as instituições financeiras através de APIs padronizadas**.

### 🔄 Antes vs. Depois do Open Finance

```text
ANTES (integrações tradicionais):

┌──────┐   CNAB 240    ┌──────┐
│ Itaú │ ────────────→  │      │
└──────┘                │      │
┌──────┐   CNAB 400    │      │
│ BB   │ ────────────→  │ ERP  │   ← Cada banco = 1 integração
└──────┘                │      │      específica, formato diferente,
┌──────┐   OFX         │      │      manutenção independente
│ Brad │ ────────────→  │      │
└──────┘                │      │
┌──────┐   CSV          │      │
│ Sant │ ────────────→  │      │
└──────┘                └──────┘

DEPOIS (Open Finance):

┌──────┐                ┌──────────────┐              ┌──────┐
│ Itaú │ ──┐            │              │   API REST   │      │
└──────┘   │  APIs      │ Open Finance │   JSON       │      │
┌──────┐   ├──padroni-→ │ (ou Pluggy/  │ ──────────→  │ ERP  │
│ BB   │ ──┤  zadas     │  Belvo como  │   Formato    │      │
└──────┘   │            │  agregador)  │   único      │      │
┌──────┐   │            │              │              │      │
│ Brad │ ──┤            └──────────────┘              └──────┘
└──────┘   │
┌──────┐   │
│ Sant │ ──┘
└──────┘
```

### 📋 Benefícios concretos para a conciliação

#### 1. API única para todos os bancos

Em vez de manter N integrações com N formatos diferentes, o software de tesouraria consome **uma única API padronizada** que retorna dados de qualquer instituição participante do Open Finance.

- **Formato padronizado**: JSON com estrutura definida pelo Banco Central
- **Sem CNAB**: elimina a complexidade de parsear arquivos CNAB 240/400 com layouts que variam por banco
- **Sem VAN/SFTP**: comunicação direta via HTTPS/REST

#### 2. Dados em tempo real (ou near real-time)

As APIs do Open Finance permitem consultar saldos e transações com atualização muito mais frequente do que os arquivos CNAB tradicionais, que tipicamente são processados uma ou duas vezes ao dia.

```text
Tradicional (CNAB):
  06:00  ─── Banco gera arquivo ───→ 08:00 ─── Importação no ERP ───→ 09:00 ─── Conciliação
  (transações do dia anterior)        (D+1, melhor caso)

Open Finance (API):
  A qualquer momento ─── GET /accounts/{id}/transactions ───→ Dados atualizados
  (consulta sob demanda, múltiplas vezes ao dia)
```

Isso permite uma **conciliação contínua** ao longo do dia, em vez de um processo batch executado uma vez por dia ou por semana.

#### 3. Dados enriquecidos e padronizados

As APIs do Open Finance retornam informações **mais ricas e estruturadas** do que os arquivos CNAB tradicionais:

| Dado | CNAB tradicional | Open Finance API |
|---|---|---|
| Descrição da transação | Texto livre, truncado, sem padrão | Campos estruturados (tipo, categoria, contraparte) |
| Identificação da contraparte | Parcial ou ausente | CPF/CNPJ, nome, instituição |
| Tipo de transação | Código numérico do banco | Enumeração padronizada |
| Categorização | Inexistente | Disponível via enrichment (ex: [[pluggy-open-finance-api|Pluggy]]) |
| Referência cruzada | Número do documento (quando presente) | Identificadores padronizados |

Esses dados mais ricos permitem **regras de matching mais inteligentes**, reduzindo a quantidade de exceções que precisam de tratamento manual.

#### 4. Cobertura ampla de instituições

Com o Open Finance, o software de tesouraria ganha acesso a **todas as instituições participantes** — não apenas os 4-5 grandes bancos para os quais integrações CNAB foram desenvolvidas. Isso é especialmente relevante para:

- Empresas que utilizam **bancos digitais** (Inter, Cora, C6, etc.)
- Empresas com contas em **cooperativas** (Sicredi, Sicoob)
- **Pequenas empresas** que usam fintechs como banco principal

#### 5. Redução do custo de manutenção

| Aspecto | Integrações tradicionais | Open Finance |
|---|---|---|
| **Custo por novo banco** | Semanas de desenvolvimento | Já incluso no padrão |
| **Manutenção de layouts** | Constante (bancos mudam formatos) | Versionamento centralizado pelo BCB |
| **Suporte a novos produtos** | Requer desenvolvimento específico | Novos produtos incluídos nas APIs |
| **Certificação** | Não aplicável | Certificação FAPI obrigatória |

---

## 🏗️ Arquitetura de conciliação automatizada com Open Finance

### Modelo para software de tesouraria

```text
┌─────────────────────────────────────────────────────────────────────┐
│                     Software de Tesouraria                          │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              Motor de Conciliação                            │   │
│  │                                                              │   │
│  │  1. Coleta     → Busca transações via Open Finance API       │   │
│  │  2. Normaliza  → Padroniza dados internos e bancários        │   │
│  │  3. Matching   → Aplica regras de correspondência            │   │
│  │  4. Exceções   → Enfileira itens não conciliados             │   │
│  │  5. Resolução  → Workflow humano para exceções               │   │
│  │  6. Ajuste     → Gera lançamentos contábeis automáticos      │   │
│  │  7. Relatório  → Evidência para auditoria                    │   │
│  └─────────────────────────┬───────────────────────────────────┘   │
│                             │                                       │
│  ┌──────────────────────────▼──────────────────────────────────┐   │
│  │           Camada de Integração Open Finance                  │   │
│  │                                                              │   │
│  │  - Pluggy / Belvo / API direta                               │   │
│  │  - Gestão de consentimentos                                  │   │
│  │  - Cache e sincronização de dados                            │   │
│  │  - Fallback para CNAB (bancos não cobertos)                  │   │
│  └──────────────────────────┬──────────────────────────────────┘   │
│                             │                                       │
└─────────────────────────────┼───────────────────────────────────────┘
                              │
               ┌──────────────▼──────────────┐
               │   Instituições Financeiras   │
               │   via Open Finance APIs      │
               │                              │
               │  Itaú · Bradesco · BB        │
               │  Santander · Caixa · Inter   │
               │  Sicredi · Sicoob · +outros  │
               └─────────────────────────────┘
```

### Regras de matching automático

O motor de conciliação pode aplicar regras em camadas, do mais preciso ao mais flexível:

```text
Nível 1 — Match exato
  Valor + Data + Referência (ID do documento)
  → Conciliação automática, confiança alta

Nível 2 — Match por valor e data
  Valor exato + Data (±1 dia útil)
  → Conciliação automática com flag de revisão

Nível 3 — Match por valor aproximado
  Valor com tolerância (ex: ±R$ 0,05 para arredondamentos)
  + Data (±3 dias úteis)
  → Sugestão de matching, requer confirmação

Nível 4 — Match por agrupamento
  Soma de múltiplos lançamentos internos = 1 crédito bancário
  (ex: múltiplos boletos pagos por um cliente em lote)
  → Sugestão de matching N:1, requer confirmação

Nível 5 — Não conciliado
  Nenhuma regra encontrou correspondência
  → Entra na fila de exceções para tratamento manual
```

### Tipos de exceção e tratamento

| Tipo de exceção | Causa típica | Ação recomendada |
|---|---|---|
| **Débito no banco sem registro interno** | Tarifa bancária, débito automático, IOF | Criar lançamento contábil |
| **Crédito no banco sem registro interno** | Depósito não identificado, rendimento | Identificar origem, registrar recebimento |
| **Registro interno sem correspondência no banco** | Pagamento ainda não processado, cheque não compensado | Aguardar processamento ou investigar |
| **Diferença de valor** | Juros, multa, desconto, IOF | Registrar ajuste com a diferença |
| **Diferença de data** | Float bancário, feriado, horário de corte | Ajustar data de efetivação |

---

## 📊 Impacto mensurável da automação

### Métricas de referência

| Indicador | Processo manual | Semi-automatizado | Open Finance + automação |
|---|---|---|---|
| **Tempo de conciliação** (por conta/mês) | 4-8 horas | 1-2 horas | 10-30 minutos |
| **Taxa de matching automático** | 0% | 60-75% | 85-95% |
| **Defasagem dos dados** | D+1 a D+5 | D+1 | Near real-time |
| **Custo de integração por novo banco** | 2-6 semanas dev | 1-3 semanas dev | 0 (já coberto) |
| **Erros humanos** | Alto | Moderado | Baixo |
| **Cobertura de instituições** | 3-5 bancos | 5-10 bancos | Todas do Open Finance |

### Exemplo prático

> [!example] Cenário: empresa média com 5 bancos e 3.000 transações/mês
>
> **Antes** (manual com CNAB):
> - 1 analista dedicado ~3 dias/mês para conciliação
> - Conciliação feita 1x/mês no fechamento
> - ~20% das transações requerem investigação manual
> - Posição de caixa confiável somente após fechamento
>
> **Depois** (Open Finance + motor de conciliação):
> - Conciliação contínua, múltiplas vezes ao dia
> - ~90% das transações conciliadas automaticamente
> - Analista atua apenas nas exceções (~300 itens/mês em vez de 3.000)
> - Posição de caixa confiável em tempo real
> - Fechamento mensal reduzido de 3 dias para 4 horas

---

## 🔑 Papel dos agregadores (Pluggy, Belvo) na conciliação

Empresas que desenvolvem software de tesouraria podem optar por **consumir as APIs do Open Finance diretamente** ou usar um **agregador como a [[pluggy-open-finance-api|Pluggy]]**. As diferenças:

| Aspecto | API direta do Open Finance | Via agregador (Pluggy/Belvo) |
|---|---|---|
| **Complexidade de integração** | Alta — exige certificação FAPI, ICP-Brasil, registro no diretório | Baixa — SDK pronto, integração em dias |
| **Manutenção** | Por conta da empresa | Por conta do agregador |
| **Custo** | Infraestrutura própria | Assinatura mensal |
| **Enriquecimento de dados** | Não incluso | Categorização, insights |
| **Conectores adicionais** | Apenas Open Finance regulado | Open Finance + conectores diretos |
| **Iniciação de pagamento** | Requer autorização ITP | Incluso (Pluggy é ITP autorizado) |
| **Time-to-market** | Meses | Semanas |

> [!tip] Recomendação para software de tesouraria
> Para empresas que estão **começando** a integrar Open Finance, usar um agregador como a Pluggy acelera drasticamente o time-to-market. A migração para API direta pode ser considerada posteriormente, quando o volume justificar o investimento em infraestrutura própria.

---

## 🗺️ Evolução esperada

### Curto prazo (2026)

- Expansão da cobertura de bancos PJ no Open Finance
- Melhoria na qualidade e padronização dos dados transacionais
- Pix Automático viabilizando conciliação de recorrências

### Médio prazo (2027-2028)

- **Conciliação preditiva**: IA antecipando matches antes da efetivação bancária
- **Conciliação cross-entity**: consolidação automática para grupos econômicos
- **Integração com Open Insurance e Open Investment**: conciliação ampliada para seguros e investimentos

### Longo prazo

- **Conciliação contínua e invisível**: o conceito de "fechar a conciliação" desaparece — tudo é conciliado em tempo real, automaticamente
- **Zero-touch reconciliation**: exceções tratadas por IA com supervisão humana apenas para casos críticos

---

## 📚 Notas relacionadas

- [[open-finance-brasil]] — Pesquisa completa sobre Open Finance: definição, regulação, impacto e oportunidades
- [[pluggy-open-finance-api]] — Infraestrutura da Pluggy para Open Finance, incluindo produto ERP Banking com conciliação
