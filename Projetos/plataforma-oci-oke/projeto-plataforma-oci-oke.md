---
title: "Plataforma OCI OKE"
date: 2026-04-23
tags:
  - projeto
  - oci
  - oke
  - plataforma
  - status/rascunho
area: execucao
status: em-andamento
owner:
---

# Plataforma OCI OKE

## Objetivo

Estruturar a plataforma de execucao em OCI OKE para suportar deploy, rede, observabilidade, controle de acesso e migracao de aplicacoes com padrao operacional reproducivel.

## Escopo

- Criacao e configuracao de cluster OKE
- Conectividade de rede com ambiente existente
- Acesso ao OCIR
- Estrategia de deploy e versionamento de imagens
- Monitoramento e componentes de plataforma

## Status atual

Documentacao operacional consolidada e migrada para a estrutura principal do vault. Falta transformar a trilha em execucao e decisoes formais quando necessario.

## Runbooks principais

- [[deploy-oke]]
- [[configurando-acesso-ocir]]
- [[conceder-acesso-usuario-oke]]
- [Trilha criar cluster OKE](./criando-cluster-oke/README.md)

## Arquitetura relacionada

- [[arquitetura-rede-oke-xrt-interno]]
- [[estrategia-versionamento-imagens-oci-oke]]

## Riscos e dependencias

- Dependencia de IAM e permissoes corretas no OCI
- Dependencia de conectividade entre VCNs
- Exposicao de segredos operacionais se a documentacao nao for higienizada
- Necessidade de padronizar rollout entre dev e homologacao

## Proximos passos

- Converter decisoes relevantes em ADRs
- Criar versoes revisadas dos runbooks com menos redundancia
- Definir se esta trilha continua como projeto ativo ou vira documentacao permanente de plataforma
