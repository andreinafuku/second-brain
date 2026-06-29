---
title: "Maker-Checker"
date: 2026-06-18
tags:
  - glossario
  - seguranca
  - compliance
  - status/rascunho
area: arquitetura
aliases:
  - Maker-Checker
  - Maker Checker
  - Dupla Autorizacao
  - Dupla Autorização
---

# Maker-Checker

## 📌 Definição

Controle de segurança em que **uma pessoa cria/inicia** uma operação (*maker*) e **outra, diferente, aprova** (*checker*) antes que ela tenha efeito. Também chamado de **dupla autorização** / segregação de funções (*segregation of duties*).

## 🔎 Contexto de uso

Comum em sistemas financeiros e ERPs para ações sensíveis (concessão de acesso, pagamentos, mudanças de privilégio), onde nenhuma pessoa deve concentrar poder de executar e aprovar sozinha. Reforça auditoria e reduz risco de fraude/erro.

## 💡 Exemplo prático

No novo sistema da XTPG, **toda concessão de acesso a empresas passa pelo Admin** e exige maker-checker: o administrador cria a concessão (define o nó da hierarquia, validade) e a envia **para aprovação**; um segundo usuário aprova antes de o acesso valer. Isso elimina o antigo papel do supervisor concedendo acesso diretamente — alinhado a boas práticas de ERP e a clientes com auditoria forte sobre privilégios financeiros (a pagar, a receber, tesouraria).

## Termos relacionados

- [[pii-personally-identifiable-information]]

## Fontes

- [[2026-06-10]] — conversa com Mota (Usuários e Segurança)
