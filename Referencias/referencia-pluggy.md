---
title: "Pluggy - Agregador Open Finance"
date: 2026-06-18
tags:
  - referencia
  - financeiro
  - integracao
  - status/rascunho
tipo: produto/plataforma
autor: Pluggy
url: https://pluggy.ai
aliases:
  - Pluggy
---

# Pluggy - Agregador Open Finance

## Dados da fonte

| Campo  | Valor                 |
| ------ | --------------------- |
| Tipo   | Produto / Plataforma  |
| Autor  | Pluggy                |
| URL    | https://pluggy.ai     |
| Acesso | 2026-06-18            |

## Resumo

Plataforma brasileira de agregação de dados financeiros via [[open-finance]]. Já **certificada junto ao Banco Central**, atua como **intermediário**: expõe APIs para conectar contas, saldos, extratos, cartões e investimentos de vários bancos, poupando o cliente de se certificar diretamente no BC.

## Pontos-chave

- Tratada no projeto XTPG como **subdomínio genérico** ([[ddd-domain-driven-design]]): em vez de construir/certificar a conexão bancária, faz-se parceria com a Pluggy.
- Oferece **dois modos de conexão**: via Open Finance (com tela de consentimento do próprio banco) e **integração direta** com alguns bancos (sem a tela de consentimento padrão, com fluxo de autenticação próprio).
- Cada conta recebe um **ID interno da Pluggy** usado para amarrar saldos e transações.
- Possui **conta trial (~15 dias)** para testar o plugin/conexão.
- Existe a skill `pluggy-api-docs` neste ambiente para implementar/depurar integração com a API.

> [!important] Decisão de arquitetura
> O ID da Pluggy **não** deve virar identificador interno das nossas contas — apenas campo informativo. Isso evita amarração ao fornecedor e permite trocar de agregador isolando `BankConnectivity` de `BankTransaction`. Ver [[load-bearing]] (US INVEST-008, o de-para de IDs).

## Como se conecta ao meu contexto

Agregador escolhido para a integração Open Finance do novo sistema da XTPG (substituto do [[referencia-universe|Universe]]). Habilita trazer banco/agência/conta automaticamente para clientes com milhares de contas (ex.: Rede D'Or, Odebrecht), sem cadastro manual.

## Notas relacionadas

- [[open-finance]]
- [[ddd-domain-driven-design]]
- [[load-bearing]]
- [[2026-06-18]]
