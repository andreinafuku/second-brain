---
title: "Pluggy — Infraestrutura de Open Finance no Brasil"
tags:
  - pesquisa
  - arquitetura/integracoes
  - referencia
  - status/consolidado
date: 2026-04-13
area: arquitetura
---

# 🔌 Pluggy - Infraestrutura de Open Finance no Brasil

## 🏢 Visão Geral da Empresa

**Pluggy** (https://www.pluggy.ai/) é uma fintech brasileira fundada em 2019, focada em desenvolvedores, que fornece infraestrutura de **Open Finance** através de uma API única. A empresa é uma **Instituição de Pagamento autorizada pelo Banco Central do Brasil** (CNPJ 37.943.755/0001-30), operando como **Iniciador de Transação de Pagamento (ITP)** sob a Resolução BCB nº 80.

### Dados Corporativos

- **Razão Social**: Pluggy Brasil Instituição de Pagamento LTDA
- **Fundação**: 2019
- **Sede**: Brasil
- **Certificação**: ISO 27001
- **Funding total**: ~US$ 10,7M
- **Investidores**: Y Combinator, Fenasbac, Gaingels, Iluminar Ventures, Lift Lab, B Venture Capital
- **Aceleradoras/Parceiros**: Y Combinator, Liga Ventures, Plug and Play, Lift, FIDinsiders, Open Startups, Banking Transformation
- **Parceria estratégica**: B3 (Bolsa de Valores do Brasil)

### Missão

Ativar o potencial do Open Finance, permitindo que empresas compreendam o comportamento financeiro de seus clientes, realizem movimentações financeiras e desenvolvam produtos inovadores.

---

## 🛠️ Produtos e Serviços

### 1. Pluggy API (Core)

API central que conecta contas financeiras de usuários em múltiplas instituições, retornando dados padronizados:

- **Histórico de transações** (com categorização)
- **Saldos de contas** (corrente, poupança)
- **Portfólios de investimentos** (posições e transações)
- **Dados de identidade** (nome, CPF, contatos, endereço)
- **Informações de cartão de crédito** (parcelas, limites, datas de vencimento, pagamento mínimo)
- **Dados de empréstimos** (saldo, termos, condições)
- **Dados de pagamento** (boletos, PIX)

### 2. Pluggy Connect Widget

Solução no-code/low-code para conexão de contas bancárias. Disponível para:

- React
- React Native
- Next.js
- JavaScript puro

Permite personalização visual e integração rápida sem complexidade de implementação.

### 3. Iniciação de Pagamentos

- **PIX instantâneo** dentro de aplicações
- **PIX Agendado** (pagamentos futuros programados)
- **PIX Protestável** (com capacidade de protesto)
- **PIX Split** (pagamento distribuído)
- **Boletos** (geração e processamento)
- **Pagamentos em lote** (uma autorização, múltiplos pagamentos)

### 4. PIX Automático

Transforma cobranças recorrentes em receita automatizada. Permite débitos automáticos via PIX entre quaisquer bancos. Funcionalidades em beta:

- Agendamento automático
- Mecanismos de retry automático

### 5. Data Enrichment (Enriquecimento de Dados)

Converte dados financeiros brutos em insights acionáveis:

- Categorização automática de transações
- Análise de pagamentos recorrentes
- Insights de conexão (análise em nível de item)

### 6. ERP Banking

Solução específica para plataformas ERP, integrando dados financeiros, cobranças e pagamentos:

- **Conciliação bancária automatizada** (eliminando tarefas manuais e erros)
- **Detalhes de transações em tempo real** (matching entre extratos e contabilidade)
- **Smart Account**: regras de negócio para movimentação automática de caixa via PIX
- **Smart Transfers**: gestão automatizada de fundos entre contas da mesma titularidade
- **Batch Payments**: uma autorização para múltiplos pagamentos

### 7. Portabilidade de Previdência

Experiências automatizadas e personalizadas para transferência de previdência.

### 8. Portabilidade de Crédito

Soluções simplificadas para portabilidade de crédito.

### 9. Meu Pluggy

Ferramenta de gestão de consentimento do usuário para operações de Open Finance.

---

## ⚙️ Capacidades Técnicas

### APIs Reguladas (Open Finance)

Dados disponíveis através das APIs reguladas pelo Banco Central:

| Categoria | Dados Disponíveis |
|-----------|------------------|
| **Identidade** | Nome, CPF, contatos, endereço, parentes |
| **Contas** | Saldos, limites, extratos categorizados (corrente e poupança) |
| **Cartões de Crédito** | Parcelas, limites, datas de fechamento/vencimento, pagamento mínimo |
| **Transações** | Comportamento financeiro derivado de dados de transações |

### APIs de Inteligência

- **Connection Insights**: análise em nível de item/conexão
- **Transaction Enrichment**: enriquecimento e categorização de transações
- **Recurring Payments Analysis**: análise de pagamentos recorrentes

### SDKs e Ferramentas para Desenvolvedores

| Plataforma | Disponibilidade |
|-----------|----------------|
| **Node.js** | SDK oficial (`pluggy-sdk` no npm, v0.74.0+) |
| **.NET** | SDK oficial |
| **Java** | SDK oficial |
| **Python** | SDK auto-gerado (`pluggy_sdk` no PyPI) |
| **React/React Native/Next.js** | Connect Widget |
| **Postman** | Coleção "Run in Postman" |
| **Bubble** | Integração no-code |
| **Webhooks** | Sincronização bidirecional |

### Conectores Suportados

#### Bancos Pessoa Física (9+ instituições)

| Instituição | Produtos Suportados |
|------------|-------------------|
| Caixa Econômica Federal | Identity, Accounts, Credit Card, Transactions, PaymentData, Investments |
| Banco Inter | Identity, Accounts, Credit Card, Transactions, Investments |
| Itaú Cartões | Identity, Credit Card |
| Mercado Pago | Identity, Accounts, Transactions |
| Safra Bank | Accounts, Transactions, Investments |
| Wise | Identity, Accounts, Transactions |
| Brasilprev | Identity, Investments |
| Caixa Previdência | Identity, Investments |
| Lemon Cash | Accounts |
| Ethereum Networks | Investments |

#### Bancos Pessoa Jurídica (11+ instituições)

- Santander Empresas
- Bradesco Empresas
- Itaú Empresas
- Caixa Econômica Federal Empresas
- Banco do Brasil Empresas
- Banco Inter Empresas
- Sicredi Empresas
- Sicoob PJ / Sicoob API
- Itaú BBA
- Conta Azul
- Cora
- Semear
- Efí Bank

#### Plataformas de Investimento (11 instituições)

- XP Investimentos
- BTG Pactual
- Empiricus Investimentos
- Eqi
- Avenue
- XP - Wealth
- Mercado Bitcoin
- BTG - Wealth
- Ágora Investimentos
- Clear Corretora
- Necton

#### Economia Digital

- Deel (Identity, Accounts)
- OnTop (Identity, Accounts)
- Splitwise (Identity, Accounts, Transactions)

> [!info] Conector dinâmico
> A lista completa e atualizada pode ser obtida via endpoint `/connectors?isOpenFinance=true`. Novos conectores são adicionados automaticamente. Cada conector pode suportar diferentes produtos.

### Documentação e Recursos

- **Docs**: https://docs.pluggy.ai/
- **Onboarding**: Quickstart em 5 minutos
- **API Reference**: Documentação completa no padrão ReadMe
- **Comunidade**: Discord
- **Status**: Monitoramento de serviço disponível
- **Changelog**: Atualizações de novos conectores e features
- **Especificação OpenAPI**: Disponível (base para SDKs auto-gerados)

---

## 💰 Modelo de Negócio e Preços

### Planos

| Plano | Preço | Características |
|-------|-------|----------------|
| **Free Trial** | R$ 0 (14 dias) | Acesso completo à API, até 20 contas conectadas, suporte básico |
| **Basic** | A partir de R$ 2.500/mês | Open Finance + conexões diretas, Help Desk, Widget customizável, Iniciação de pagamento |
| **Enterprise** | Sob consulta | Volume adaptado, suporte premium, integrações customizadas, acesso a produtos beta |

### Modelo de Cobrança

- Baseado em assinatura mensal
- Possível cobrança por volume de conexões/requisições (detalhes sob consulta para Enterprise)
- Todos os planos incluem: API, suporte técnico, infraestrutura LGPD-compliant

---

## 🎯 Mercado-Alvo

### Segmentos Primários

1. **Plataformas ERP** transformando-se em ecossistemas financeiros
2. **Fintechs** que precisam de integração Open Finance
3. **Empresas de contabilidade e BPO financeiro**
4. **Processadores de pagamento e plataformas de crédito**
5. **Desenvolvedores** construindo produtos financeiros

### Modelo: B2B e B2B2C

A Pluggy vende para empresas (B2B) que por sua vez utilizam a infraestrutura para servir seus clientes finais (B2B2C).

### Clientes Confirmados

| Cliente | Segmento |
|---------|----------|
| **Conta Azul** | ERP / Gestão Financeira |
| **Nibo** | Contabilidade / BPO Financeiro |
| **MarketUP** | ERP |
| **Kamino** | Gestão Financeira |
| **Cloud Gym** | Gestão de Academias |
| **Grupo Boticário** | Varejo / Cosméticos |
| **Linx** | Software para Varejo |
| **Koin** | Pagamentos / Crédito |

---

## 🏆 Cenário Competitivo

### Principais Concorrentes no Brasil

| Empresa | Foco | Diferencial |
|---------|------|-------------|
| **Belvo** | Open Finance, Dados e Pagamentos | +8M consentimentos únicos, entre os 3 maiores (ao lado de Nubank e Mercado Pago) |
| **Quanto** | Open Finance | Autorizado para iniciação de pagamento |
| **Klavi** | Open Finance, Dados | Aguardando autorização ITP do BC |
| **Finansystech (Celcoin)** | Open Finance | Infraestrutura financeira ampla |
| **Akropoli** | Open Finance | Foco em dados |
| **Lina Open X** | Open Finance | Plataforma de dados |

### Diferenciais da Pluggy

- **Autorização ITP pelo Banco Central** (obtida em junho/2024)
- **Certificação ISO 27001**
- **Y Combinator alumni** (credibilidade internacional)
- **Parceria com B3**
- **Foco em developer experience** (SDKs múltiplos, docs de qualidade, onboarding rápido)
- **Produto ERP Banking** específico para plataformas de gestão
- **PIX Automático** como diferencial de produto

---

## 🏦 Oportunidades para Empresa de Tesouraria (Treasury)

> [!tip] Análise Estratégica
> Como uma empresa que desenvolve software de gestão de tesouraria para grandes, médias e pequenas empresas pode alavancar a infraestrutura da Pluggy.

### 1. 📊 Agregação Multi-Banco para Dashboards de Tesouraria

**Problema**: Empresas mantêm contas em múltiplos bancos e precisam de visão consolidada.

**Solução com Pluggy**:
- API única conecta a 30+ instituições financeiras (PJ)
- Saldos em tempo real de todas as contas
- Extratos consolidados e padronizados
- Elimina necessidade de múltiplas integrações bancárias proprietárias (CNAB, APIs individuais)

**Valor gerado**: Dashboard unificado de posição de caixa em tempo real, sem depender de arquivos bancários ou integrações one-to-one com cada banco.

### 2. 💸 Iniciação de Pagamentos para Operações de Tesouraria

**Problema**: Tesouraria precisa realizar pagamentos a fornecedores, folha, impostos etc.

**Solução com Pluggy**:
- **PIX instantâneo** para pagamentos urgentes
- **PIX Agendado** para programação de pagamentos futuros
- **Batch Payments** para lotes de pagamento (fornecedores, folha)
- **Boletos** para pagamentos tradicionais
- **Smart Transfers** para movimentação entre contas da mesma empresa

**Valor gerado**: Centralização de toda a operação de pagamentos em uma plataforma, sem necessidade de acessar internet banking de múltiplos bancos.

### 3. 🔄 Conciliação Bancária Automatizada

**Problema**: Conciliação manual consome tempo e é propensa a erros.

**Solução com Pluggy**:
- Dados detalhados de transações com categorização automática
- Matching entre extratos bancários e lançamentos contábeis
- Dados em tempo real eliminam gap temporal

**Valor gerado**: Redução drástica do tempo de fechamento financeiro, de dias para horas ou minutos.

### 4. 📈 Visibilidade de Fluxo de Caixa (Cash Flow)

**Problema**: Previsão de fluxo de caixa depende de dados fragmentados.

**Solução com Pluggy**:
- **Transações categorizadas** automaticamente
- **Análise de pagamentos recorrentes** (identifica padrões)
- **Connection Insights** para análise comportamental
- Histórico de transações de múltiplas instituições

**Valor gerado**: Previsão de fluxo de caixa mais precisa, baseada em dados reais e padrões identificados automaticamente.

### 5. 🏦 Gestão Automática de Caixa (Cash Management)

**Problema**: Empresas mantêm excesso de caixa em contas não remuneradas ou com baixa rentabilidade.

**Solução com Pluggy**:
- **Smart Account**: regras automáticas de movimentação via PIX
- **Smart Transfers**: transferências automáticas entre contas da mesma titularidade
- Visibilidade consolidada de saldos para otimização

**Valor gerado**: Otimização automática da posição de caixa, maximizando rendimentos e minimizando custos bancários.

### 6. 📋 Análise de Crédito e Working Capital

**Problema**: Empresas precisam de análise de crédito para clientes/fornecedores ou para otimizar capital de giro.

**Solução com Pluggy**:
- **Dados de identidade verificados** em instituições financeiras
- **Histórico financeiro** completo (contas, investimentos, crédito)
- **Dados de empréstimos** existentes
- **Enriquecimento de dados** para scoring

**Valor gerado**: Análise de risco baseada em dados financeiros reais, não apenas em bureau de crédito. Possibilidade de oferecer antecipação de recebíveis ou crédito com melhor precificação.

### 7. 🔗 Modelo de Integração Proposto

```
┌──────────────────────────────────┐
│    Software de Tesouraria        │
│  ┌───────────────────────────┐   │
│  │  Dashboard Consolidado    │   │
│  │  - Posição de caixa       │   │
│  │  - Fluxo de caixa         │   │
│  │  - Conciliação            │   │
│  │  - Pagamentos             │   │
│  └───────────┬───────────────┘   │
│              │                    │
│  ┌───────────▼───────────────┐   │
│  │    Pluggy API Layer       │   │
│  │  - Connect Widget         │   │
│  │  - Data Aggregation       │   │
│  │  - Payment Initiation     │   │
│  │  - Data Enrichment        │   │
│  └───────────┬───────────────┘   │
└──────────────┼───────────────────┘
               │
    ┌──────────▼──────────┐
    │  Open Finance / APIs │
    │  - Itaú Empresas     │
    │  - Bradesco Empresas │
    │  - BB Empresas       │
    │  - Santander Empresas│
    │  - Caixa Empresas    │
    │  - +25 instituições  │
    └─────────────────────┘
```

### 8. 💡 Oportunidades de Produto Derivadas

| Oportunidade | Descrição | Complexidade |
|-------------|-----------|-------------|
| **Treasury Dashboard** | Visão consolidada multi-banco com saldos em tempo real | Média |
| **Payment Hub** | Central de pagamentos (PIX, boleto, lote) integrada | Média-Alta |
| **Auto-Reconciliation** | Conciliação automática com ERP/contabilidade | Média |
| **Cash Flow Forecasting** | Previsão baseada em dados reais + ML | Alta |
| **Smart Cash Pool** | Movimentação automática entre contas para otimização | Média |
| **Credit Scoring** | Análise de risco com dados Open Finance | Alta |
| **Supplier Finance** | Antecipação de pagamentos a fornecedores | Alta |
| **Multi-entity Consolidation** | Visão consolidada para grupos empresariais | Média |

### 9. ⚠️ Considerações e Riscos

- **Custo**: Plano básico a partir de R$ 2.500/mês — precifica-se por volume
- **Dependência**: Infraestrutura crítica depende de terceiro (Pluggy)
- **Cobertura PJ**: Nem todos os bancos PJ estão disponíveis (validar lista atualizada)
- **Consentimento**: Open Finance requer consentimento ativo do usuário final (renovação periódica)
- **Rate Limits**: Existem limites de requisições tanto para conectores regulados quanto diretos
- **SLA**: Validar SLA de disponibilidade para uso em ambiente de produção de tesouraria

---

## 🔗 Links e Referências

- Site oficial: https://www.pluggy.ai/
- Documentação API: https://docs.pluggy.ai/
- Cobertura de conectores: https://docs.pluggy.ai/docs/connectors-coverage
- Open Finance regulado: https://docs.pluggy.ai/docs/open-finance-regulated
- GitHub (Node SDK): https://github.com/pluggyai/pluggy-node
- ERP Banking: https://www.pluggy.ai/en/erp
- Preços: https://www.pluggy.ai/pricing

---

## 📌 Notas Relacionadas

- [[open-finance-brasil]] — Pesquisa completa sobre Open Finance: definição, regulação, impacto e oportunidades
- [[conciliacao-bancaria]] — Conciliação bancária: conceito, importância e automação com Open Finance
- [[ai-gateway-arquitetura-financeiro]] — Arquitetura de AI Gateway para sistema financeiro
- [[estrategias-modernizacao-sistemas]] — Estratégias de modernização de sistemas
