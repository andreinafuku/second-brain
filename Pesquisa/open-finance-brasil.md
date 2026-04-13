---
title: "Open Finance Brasil — Pesquisa Completa"
tags:
  - pesquisa
  - arquitetura/open-finance
  - referencia
  - status/consolidado
date: 2026-04-13
area: arquitetura
---

# 🏦 Open Finance Brasil — Pesquisa Completa

## 📌 O que é Open Finance?

**Open Finance** (sistema financeiro aberto) é um modelo regulatório e tecnológico que permite o **compartilhamento padronizado de dados e serviços financeiros** entre instituições autorizadas, mediante consentimento explícito do cliente. O cliente é o **dono dos seus dados**, não o banco.

### 🔀 Diferença entre Open Banking e Open Finance

| Aspecto | Open Banking | Open Finance |
|---|---|---|
| **Escopo** | Dados bancários tradicionais (contas, transações, crédito) | Amplo: bancos + seguros + investimentos + previdência + câmbio |
| **Instituições** | Bancos e instituições de pagamento | Bancos, fintechs, seguradoras, corretoras, cooperativas, empresas de câmbio, fundos de previdência |
| **Serviços** | Compartilhamento de dados e informações bancárias | Compartilhamento de dados + iniciação de pagamentos + portabilidade de crédito + Open Insurance + Open Investment |

O Open Finance é uma **evolução do Open Banking**, ampliando o escopo para todo o sistema financeiro. O Brasil **renomeou oficialmente** o programa de "Open Banking Brasil" para "Open Finance Brasil" em março de 2022.

### 🌍 Contexto global

- **Reino Unido**: Pioneiro com Open Banking desde 2018 (CMA Order)
- **União Europeia**: PSD2 (Payment Services Directive 2) desde 2018
- **Austrália**: Consumer Data Right (CDR) desde 2020
- **Brasil**: Um dos modelos **mais completos do mundo**, abrangendo não só banking mas todo o ecossistema financeiro (seguros, investimentos, previdência, câmbio)

> [!tip] O Brasil é referência mundial
> O modelo brasileiro é considerado um dos mais ambiciosos e completos do planeta, com mais de **100 milhões de clientes conectados** e **128 milhões de consentimentos ativos** (janeiro de 2026).

---

## ⚖️ Regulação no Brasil

### 🏛️ Quem regula

O **Banco Central do Brasil (BCB)** é o principal regulador, em conjunto com o **Conselho Monetário Nacional (CMN)**. Também participam:
- **CVM** (Comissão de Valores Mobiliários) — para investimentos
- **SUSEP** (Superintendência de Seguros Privados) — para seguros e previdência
- **AB Fintechs** — associação representativa

### 📜 Principais atos normativos

| Normativo | Data | Descrição |
|---|---|---|
| **Resolução Conjunta nº 1/2020** | 04/05/2020 | Marco regulatório fundador — dispõe sobre a implementação do Open Finance por instituições financeiras e de pagamento autorizadas pelo BCB |
| **Resolução BCB nº 32/2020** | 29/10/2020 | Estabelece requisitos técnicos e procedimentos operacionais para implementação |
| **Instrução Normativa BCB nº 305/2022** | 15/09/2022 | Requisitos operacionais complementares |
| **Resolução Conjunta nº 10/2024** | 04/07/2024 | Altera a Resolução Conjunta nº 1/2020 — amplia critérios de participação obrigatória e escopo de compartilhamento |
| **Resolução BCB nº 400/2024** | 04/07/2024 | Diretrizes para a Estrutura de Governança definitiva do Open Finance |
| **Resolução BCB nº 517/2025** | 2025 | Requisitos para conformidade FAPI em produção |
| **Instrução Normativa BCB nº 720/2026** | 02/04/2026 | Atualizações normativas recentes |

### 📅 Fases de implementação — Cronograma

#### Fase 1 — Dados abertos de produtos e serviços (30/11/2020)
- Dados públicos de produtos e serviços das instituições
- Canais de atendimento
- Produtos de contas de depósito
- Dados de correspondentes bancários
- Informações de crédito

#### Fase 2 — Compartilhamento de dados de clientes (31/05/2021)
- Compartilhamento efetivo de dados cadastrais dos clientes
- Histórico de transações em contas de depósito
- Dados de operações de crédito
- Informações de cartão de crédito
- **Mediante consentimento explícito do cliente**

#### Fase 3 — Iniciação de pagamentos e encaminhamento de propostas (30/08/2021)
- Iniciação de transações de pagamento (Pix) via terceiros
- Cliente pode receber oferta de crédito de instituição concorrente
- Encaminhamento de propostas de crédito
- Serviço de Iniciador de Transação de Pagamento (ITP)

#### Fase 4 — Expansão de dados e serviços (25/10/2021, com implementação gradual)
- **Câmbio**: Operações de câmbio
- **Seguros**: Open Insurance (seguros de pessoas e danos)
- **Investimentos**: Open Investment (CDB, Tesouro Direto, fundos de investimento, ações)
- **Previdência**: Previdência complementar aberta
- **Contas de depósito a prazo e outros produtos**

### 📊 Dados compartilhados — Escopo completo

1. **Dados bancários**: Contas correntes, poupança, transações, saldos
2. **Crédito**: Empréstimos pessoais, financiamentos, cartões de crédito, limites, taxas
3. **Investimentos**: CDB, LCI, LCA, Tesouro Direto, fundos de investimento, ações, debêntures
4. **Seguros**: Seguros de vida, auto, residencial, empresarial, viagem
5. **Previdência**: Planos de previdência complementar aberta (PGBL, VGBL)
6. **Câmbio**: Operações de câmbio, remessas internacionais
7. **Dados cadastrais**: Informações pessoais, endereço, qualificação

### 🏗️ Estrutura de Governança

A governança do Open Finance Brasil opera em **três níveis hierárquicos**:

#### 1. Órgão de Governança (Assembleia Geral)
- Participação e voto de **todas as instituições participantes**
- Aprova contas, altera estatuto, remove membros eleitos
- Número de votos proporcional à participação no financiamento, **limitado a 3% dos votos totais** por instituição
- Funciona como assembleia geral do ecossistema

#### 2. Órgão de Direção Superior (Conselho Deliberativo)
- **10 membros votantes** representando diferentes segmentos
- Delibera sobre propostas das diretorias
- Aprova padrões tecnológicos e procedimentos operacionais
- Define escopo de dados e serviços
- Aprova criação de comitês técnicos

#### 3. Diretorias (Gestão Executiva)
- Administram, gerenciam e dirigem a estrutura
- Submetem propostas orçamentárias ao Conselho
- Monitoram desempenho das instituições participantes
- Coordenam comitês técnicos
- Relacionamento com autoridades e imprensa

> [!info] Transição de governança
> Em janeiro de 2025, a governança foi transferida para a **Associação Open Finance Brasil**, uma estrutura definitiva que substituiu a governança inicial provisória, conforme Resolução BCB nº 400/2024.

### 👥 Participantes — Obrigatórios vs. Voluntários

#### Participantes obrigatórios (compartilhamento de dados)
- Instituições dos **segmentos prudenciais S1 e S2** do BCB
- Instituições pertencentes a conglomerados com **mais de 5 milhões de clientes** por dois trimestres consecutivos (a partir de 01/01/2025, conforme Resolução Conjunta nº 10/2024)
- Cobertura estimada: **~95% de todas as relações financeiras existentes** (antes eram ~51%)

#### Participantes obrigatórios (iniciação de pagamento)
- Instituições que são **participantes obrigatórios do Pix**
- Instituições detentoras de contas pertencentes a conglomerados com participantes obrigatórios do Pix
- A partir de 14/11/2024 para grandes conglomerados e cooperativas
- A partir de 02/01/2026 para todas as instituições participantes do Pix

#### Participantes voluntários
- Qualquer instituição financeira autorizada pelo BCB
- Fintechs, cooperativas menores, corretoras, seguradoras
- Podem participar como **transmissoras e/ou receptoras** de dados

### 📈 Status atual (2025-2026) — Números

| Métrica | Valor |
|---|---|
| Clientes/contas conectadas | **+100 milhões** (fev/2026) |
| Consentimentos ativos | **+128 milhões** (jan/2026), ultrapassando 160 milhões no Q1/2026 |
| Crescimento de consentimentos (2024→2025) | **+149%** |
| Crescimento de CPFs/CNPJs únicos (2024→2025) | **+143%** |
| Volume Pix via iniciação de pagamento (2025) | **R$ 15,3 bilhões** (vs R$ 3,2 bi em 2024) |
| Transações de pagamento iniciadas (2025) | **64,5 milhões** (vs 7,4 milhões em 2024) |
| Consentimentos PJ (receptoras, 2024) | ~403 mil |
| Consentimentos PJ (transmissoras, 2024) | ~407 mil |

#### Agenda 2026
- **Fevereiro/2026**: Portabilidade de crédito digital — empréstimos pessoais sem garantia e sem consignação
- **Agosto/2026**: Início dos testes de portabilidade de crédito consignado (setor público federal)
- **Novembro/2026**: Lançamento da portabilidade de crédito consignado público federal
- **2025-2026**: Regulamentação de parcerias no âmbito do Open Finance
- **Pix Automático**: Implementação prevista para junho de 2025 (pagamentos recorrentes com valores fixos ou variáveis) — implementação e certificação pausadas temporariamente, mas "Smart Transfers" (Sweeping Accounts) continuam disponíveis

---

## 🏢 Impacto na tesouraria corporativa (médias e grandes empresas)

### 💰 Visibilidade multi-banco e gestão de caixa

- **Visão consolidada**: APIs permitem agregar saldos e transações de **todas as contas bancárias** em tempo real, em um único dashboard
- **Cash visibility**: Elimina a necessidade de acessar múltiplos internet bankings manualmente
- **Cash positioning**: Posição de caixa consolidada em tempo real, incluindo diferentes bancos, moedas e subsidiárias
- **Previsão de fluxo de caixa**: Dados históricos integrados permitem modelos preditivos mais precisos

### 🔄 Conciliação bancária automatizada

- **Automação via API**: Integração direta com ERPs (SAP, Oracle, TOTVS) para conciliação automática
- Redução de intervenções manuais e erros humanos
- Identificação e correção automática de discrepâncias
- **Impacto estimado**: Redução de até 80% do tempo gasto em conciliação manual

### 💸 Iniciação de pagamentos via API

- **Pagamentos programáticos**: Execução de pagamentos (Pix, TED, boletos) diretamente via API, sem acessar internet banking
- **ITP (Iniciador de Transação de Pagamento)**: Terceiros autorizados podem iniciar pagamentos em nome da empresa
- **Checkout simplificado**: Pagamento iniciado automaticamente, cliente apenas autentica no banco
- Redução de etapas, fricção e abandono em processos de pagamento

### 📈 Acesso a crédito e financiamento

- **Compartilhamento de histórico financeiro real**: Novas instituições podem analisar dados reais da empresa para oferecer crédito
- **Melhores taxas**: Competição entre instituições baseada em dados reais, não em relacionamento
- **Portabilidade de crédito digital**: A partir de fev/2026, migração de operações entre instituições de forma 100% digital, em até 5 dias úteis
- **Capital de giro**: Acesso mais rápido e transparente a linhas de capital de giro

### 📊 Decisões financeiras baseadas em dados

- **Analytics em tempo real**: Dados financeiros atualizados para tomada de decisão
- **Integração com IA**: Dados integrados de múltiplas fontes alimentam modelos de inteligência artificial para previsões e recomendações
- **Benchmarking**: Comparação de condições (taxas, tarifas, serviços) entre instituições de forma automatizada

### 📋 Contas a pagar e a receber

- **Automatização de AP/AR**: Fluxos de pagamento e recebimento integrados via API
- **Reconciliação automática de recebíveis**: Matching automático entre faturas emitidas e pagamentos recebidos
- **Gestão de fornecedores**: Pagamentos automatizados com rastreabilidade completa
- **Antecipação de recebíveis**: Dados compartilhados facilitam operações de antecipação com melhores condições

---

## 🚀 Oportunidades para desenvolvedores de aplicações de tesouraria

### 🔗 Agregação de contas (Account Aggregation)

- **Produto**: Dashboard unificado que consolida dados de múltiplos bancos
- **Público**: Tesouraria de empresas de todos os portes
- **Valor**: Visão única de caixa, eliminando acesso manual a múltiplos internet bankings
- **Exemplo**: Plataformas como Pluggy, Belvo já oferecem infraestrutura de agregação

### 💳 Serviços de iniciação de pagamento (PISP)

- **Produto**: Plataforma que inicia pagamentos Pix/TED em nome do cliente via API
- **Público**: E-commerce, marketplaces, empresas com alto volume de pagamentos
- **Valor**: Checkout simplificado, redução de fricção, menor custo que cartão de crédito
- **Exemplo**: Mercado Pago foi o primeiro PISP no Brasil (H1 2022)

### 🏊 Cash Pooling automatizado

- **Produto**: Sistema que automatiza a concentração de caixa entre contas de diferentes bancos
- **Público**: Grandes empresas e grupos econômicos com múltiplas contas
- **Valor**: Otimização de saldos, redução de custo de capital, maximização de rendimentos
- **Funcionalidade**: Smart Transfers / Sweeping Accounts — transferências inteligentes entre contas do mesmo titular

### 🏪 Marketplace de crédito

- **Produto**: Plataforma que compara e intermedeia ofertas de crédito de múltiplas instituições
- **Público**: PMEs e grandes empresas buscando melhores condições
- **Valor**: Transparência de taxas, competição entre instituições, melhores condições
- **Funcionalidade**: Empresa compartilha dados financeiros → recebe propostas de múltiplos bancos

### 📊 Analytics financeiro em tempo real

- **Produto**: Dashboard com análises preditivas baseadas em dados de Open Finance
- **Público**: CFOs, controllers, tesoureiros
- **Valor**: Previsão de fluxo de caixa, detecção de anomalias, otimização de capital de giro
- **Tecnologia**: IA/ML alimentados por dados em tempo real de múltiplas fontes

### 🔄 Conciliação automatizada como serviço

- **Produto**: SaaS de conciliação bancária que consome dados via Open Finance
- **Público**: Empresas de todos os portes com alto volume transacional
- **Valor**: Eliminação de processos manuais, redução de erros, auditoria em tempo real

### 📱 Atendendo diferentes portes de empresa

| Porte | Necessidade principal | Oportunidade |
|---|---|---|
| **Grande empresa** | Multi-bank visibility, cash pooling, treasury analytics | Plataformas enterprise de gestão de caixa integradas |
| **Média empresa** | Conciliação, acesso a crédito, gestão de fluxo de caixa | ERPs com módulos de Open Finance integrados |
| **Pequena empresa** | Visão consolidada, crédito acessível, simplicidade | Apps simplificados de gestão financeira com Open Finance |
| **MEI/Micro** | Acesso a crédito, controle básico de caixa | Fintechs com score baseado em dados reais de Open Finance |

### 🇧🇷 Ecossistema de fintechs no Brasil

O Brasil possui **mais de 910 fintechs** em quase 40 segmentos, sendo o **maior hub fintech da América Latina**. Tendências-chave:
- Colaboração profunda entre bancos e fintechs
- Crescimento de **embedded finance** via Open Finance
- Plataformas de infraestrutura: Pluggy, Belvo, Celcoin, TecBan/Ozone

---

## 🔧 Aspectos técnicos

### 📐 Padrões de API

- **Arquitetura**: REST/JSON
- **Especificação**: OpenAPI 3.0 (Swagger)
- **Estrutura de URI**: `<host>/open-banking/<api>/<version>/<resource>`
- **Formato de dados**: JSON
- **Estrutura de request/response**: Objeto JSON com campo `data` (dados primários) e opcionalmente `meta` (paginação, metadados)
- **Repositório oficial**: [github.com/OpenBanking-Brasil/openapi](https://github.com/OpenBanking-Brasil/openapi)
- **Portal do desenvolvedor**: [openfinancebrasil.atlassian.net](https://openfinancebrasil.atlassian.net/wiki/spaces/OF/)

### 🔐 Autenticação e autorização

#### OAuth 2.0 + FAPI (Financial-grade API)

O Open Finance Brasil implementa o **Financial-grade API (FAPI) Security Profile 1.0**, que é um perfil altamente seguro construído sobre:

- **RFC 6749** — OAuth 2.0 Authorization Framework
- **RFC 6750** — Bearer Token Usage
- **RFC 7636** — PKCE (Proof Key for Code Exchange)
- **FAPI 1.0 Advanced** — Financial-grade API Part 2
- **OpenID Connect Core 1.0** — Autenticação
- **OpenID Connect Discovery** — Descoberta de endpoints (.well-known)

#### Fluxos de autenticação

- **Authorization Code Flow** com PKCE — fluxo principal
- **Client Initiated Backchannel Authentication (CIBA)** — autenticação desacoplada (dispositivo de acesso diferente do dispositivo de autenticação)
- **Pushed Authorization Requests (PAR)** — requisições de autorização enviadas diretamente ao servidor
- **response_type**: `code id_token` ou `code` com `response_mode: jwt`

#### Níveis de autenticação (LOA)

- **LOA2** (`urn:brasil:openbanking:loa2`): Autenticação de fator único
- **LOA3** (`urn:brasil:openbanking:loa3`): Autenticação multifator (mínimo dois métodos diferentes)

### 🔒 Certificados e mTLS

#### ICP-Brasil

- Certificados **ICP-Brasil** obrigatórios (infraestrutura de chaves públicas brasileira)
- ICP-Brasil emite apenas certificados **RSA x509** (sem suporte a curvas elípticas)
- Algoritmos de criptografia limitados aos **recomendados pela IANA**

#### Mutual TLS (mTLS)

- **Obrigatório** para todos os endpoints públicos
- Autenticação mútua: servidor e cliente validam certificados
- Métodos de autenticação do cliente: `oAuth2.0 mTLS` ou `private_key_jwt`
- **Dynamic Client Registration**: Registro dinâmico de clientes usando certificados

### 📝 JWT e assinatura

#### JWS (Assinatura)

- **Algoritmo obrigatório**: `PS256` (RSA-PSS com SHA-256)
- **Claims obrigatórios**: `aud`, `iss`, `jti`, `iat`
- `aud`: URL do endpoint destinatário ou organisationId do cliente
- `iss`: organisationId do remetente (do diretório)
- `jti`: UUID v4, único por clientId em janela de 86.400 segundos (24h); reuso retorna HTTP 403
- `iat`: Unix time GMT+0 com tolerância de ±60 segundos
- **JOSE Header**: `alg` (PS256), `kid`, `typ` (JWT)
- **Content-Type**: `application/jwt`
- **Validação**: Exclusivamente via JWKS do diretório

#### JWE (Criptografia)

- **Algoritmo**: `RSA-OAEP` com `A256GCM`
- Indicação de chave via header `kid`
- Headers proibidos: `x5u`, `x5c`, `jku`, `jkw`

### 🛡️ Requisitos de segurança

#### TLS

- **Cipher suites suportadas**:
  - `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
  - `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
- **TLS Session Resumption**: Desabilitado
- **TLS Renegotiation**: Desabilitado

#### Tokens de acesso

- **Expiração**: Entre 300 e 900 segundos (5 a 15 minutos)
- **Refresh tokens**: Suportados, com rotação opcionalmente desabilitável
- **ACR claim**: Sempre incluído no ID Token
- Tokens revogados quando consentimento é deletado

#### Consentimento (Consent API)

- **Escopo dinâmico**: `consent:<ConsentResourceId>` (ex: `consent:urn:bancoex:C1DD33123`)
- Access tokens emitidos **apenas quando consent status = "AUTHORIZED"**
- Recursos compartilhados apenas com consentimento válido e ativo
- Para Fase 3: Sem refresh_token quando status "CONSUMED"; emitir access_token via client credentials
- **Certificação obrigatória**: FAPI e OpenID

#### Headers de segurança

- Header `x-fapi-interaction-id` **obrigatório** em endpoints FAPI
- Rejeição de requests sem esse header

#### Tratamento de erros

- Falha de validação de assinatura: HTTP 400 com código `BAD_SIGNATURE`
- Token inválido: HTTP 401

### 🏗️ APIs disponíveis — Categorias

1. **Dados abertos (Phase 1)**: Produtos, serviços, canais de atendimento
2. **Dados cadastrais e transacionais (Phase 2)**: Contas, transações, crédito, cartões
3. **Pagamentos (Phase 3)**: Iniciação de pagamento via Pix
4. **Pagamentos automáticos**: API Automatic Payments (Pix Automático)
5. **Investimentos (Phase 4)**: CDB, fundos, ações, Tesouro Direto
6. **Seguros (Phase 4)**: Seguros de pessoas e danos
7. **Previdência (Phase 4)**: Previdência complementar aberta
8. **Câmbio (Phase 4)**: Operações cambiais
9. **Consentimento**: Gestão de consentimentos (OAuth 2.0 protected resource)
10. **Recursos**: API de recursos do diretório

---

## 🔗 Links e referências

- [Open Finance Brasil — Portal oficial](https://openfinancebrasil.org.br/)
- [Portal do desenvolvedor — Atlassian](https://openfinancebrasil.atlassian.net/wiki/spaces/OF/)
- [GitHub — OpenAPI specs](https://github.com/OpenBanking-Brasil/openapi)
- [GitHub — Specs de segurança](https://github.com/OpenBanking-Brasil/specs-seguranca)
- [BCB — Open Finance](https://www.bcb.gov.br/estabilidadefinanceira/openfinance)
- [Dashboard do cidadão — Estatísticas](https://dashboard.openfinancebrasil.org.br/)
- [Resolução Conjunta nº 1/2020 (PDF)](https://normativos.bcb.gov.br/Lists/Normativos/Attachments/51028/Res_Conj_0001_v7_L.pdf)
- [FAPI Security Profile — Draft 3](https://openfinancebrasil.atlassian.net/wiki/spaces/OF/pages/245760001)

---

## 📚 Notas relacionadas

- [[conciliacao-bancaria]] — Conciliação bancária: conceito, importância e automação com Open Finance
- [[pluggy-open-finance-api]]
- [[ai-gateway-arquitetura-financeiro]]
