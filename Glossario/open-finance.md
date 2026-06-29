---
title: "Open Finance"
date: 2026-06-18
tags:
  - glossario
  - financeiro
  - integracao
  - status/rascunho
area: arquitetura
aliases:
  - Open Finance
  - Open Banking
---

# Open Finance

## 📌 Definição

Ecossistema **regulado pelo Banco Central** que padroniza o **compartilhamento de dados bancários** entre instituições financeiras, mediante **consentimento** explícito do dono da conta. Cobre contas, saldos, extratos, cartões de crédito, investimentos e empréstimos.

## 🔎 Contexto de uso

Dois motivos principais:

1. **Para o cliente final:** não fica preso a um banco — outras instituições podem, com consentimento, ver seus dados e oferecer condições melhores (ex.: aplicações com rendimento maior, empréstimos com juros menores).
2. **Para a empresa:** padroniza a comunicação com os bancos. Antes, cada banco (Bradesco, Itaú, Banco do Brasil, Santander) tinha seu próprio formato, gerando retrabalho constante a cada mudança de campo/protocolo.

### 🔐 Consentimento

Ao conectar, o usuário é redirecionado à tela do próprio banco (internet banking, podendo exigir 2FA) para autorizar o acesso. Em conta **conjunta** pode ser necessário o consentimento de ambos os titulares; em conta **PJ**, o titular é o CNPJ, mas quem opera no dia a dia é a tesouraria/financeiro.

### 🏛️ Certificação e agregadores

Consumir as APIs do Open Finance exige certificação junto ao Banco Central — processo de segurança pesado e demorado. Por isso é comum usar um **agregador já certificado** (ex.: [[Pluggy]]) como intermediário, tratando a conexão bancária como **subdomínio genérico** (ver [[ddd-domain-driven-design]]).

## 💡 Exemplo prático

No projeto XTPG, em vez de o usuário cadastrar manualmente milhares de contas (ex.: clientes como Rede D'Or e Odebrecht), conecta-se à Pluggy via Open Finance e já se traz banco, agência e conta automaticamente.

## Termos relacionados

- [[ddd-domain-driven-design]]
- [[load-bearing]]
- [[pii-personally-identifiable-information]]

## Fontes

- [[2026-06-18]] — daily XTPG
