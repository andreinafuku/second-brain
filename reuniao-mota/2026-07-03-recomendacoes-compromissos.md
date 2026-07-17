---
title: "Conversa com Mota e Carlão - Recomendações e Compromissos - 2026-07-03"
date: 2026-07-03
tags:
  - projeto
  - lideranca
  - reuniao
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Mota
  - Carlão
---

# ✅ Conversa com Mota e Carlão — Recomendações e Compromissos (2026-07-03)

> [!info] Contexto
> Compilado das recomendações, decisões e compromissos assumidos nas duas reuniões 1:1 de
> 03/07/2026 (Andre + Mota, depois Andre + Carlão) — os indicativos de próximos passos do [[xtpg]].

## 📌 Compromissos com prazo

| Prazo | Compromisso | Responsável |
|---|---|---|
| Hoje à tarde (03/07) | Começar a publicar/criar o ambiente do Treasury Command | Andre / Clayton |
| Hoje à tarde (03/07) | Conversar com **Tayná** e **Carlão** (feedback + proposta do Atualiza) | Andre |
| Segunda ou terça (06–07/07) | Entregar ambiente de **homologação** para o Mota testar/navegar | Andre / time |
| Segunda (06/07) | Alinhar com Mota se o ambiente ficou pronto | Andre |
| Semana que vem | Novo check-in Mota × Andre (segunda ou terça) | Andre / Mota |
| Semana que vem | Mota conversa com a Tayná (após Andre) | Mota |
| A partir do dia **27** | Andre em férias (1ª semana; leva os pais para visitar o Matias) | Andre |
| Set/ago (a confirmar) | Andre em férias (2ª semana; cirurgia do ombro do filho) | Andre |

## 🌐 Ambientes e infraestrutura

- **Recomendado:** criar dois ambientes — **desenvolvimento** (Andre, Clayton, Rodrigo, Júlia) e
  **homologação** (só Mota).
- **Decidido:** homologação com **acesso restrito só ao Mota**, que usará as **contas correntes
  reais da XTPG** (não mais mocks). Fechar bem a senha.
- **Decidido:** por ora, **reusar o mesmo cluster do portal** (ambiente separado) para evitar
  ~R$600/mês de um cluster novo. Escalar/isolar depois, se necessário — é rápido.
- **Compromisso do Andre:** verificar e responder ao Mota como as **credenciais (client ID /
  client secret)** e o consentimento por cliente final ficam armazenados/identificados sob o
  guarda-chuva da XTPG.

## 🤖 Fluxo de trabalho com IA

- **Recomendado (Mota):** quando for testar, ao encontrar correções, pedir a Claude para gerar os
  cards direto no board, em vez de esperar um "dia de passar a lista" — o painel controla o fluxo
  e o time já vai "moendo".
- **Recomendado (Andre → Mota):** montar o próprio agente/CLI do Mota para gerar cards e apontar
  para o board; a instalação do CLI (inclusive do Azure DevOps) pode ser feita junto, com ajuda da
  própria IA. **Mota apoiou 100%** — objetivo comum de **reduzir tempos de espera**.
- **Recomendado:** escolher o modelo conforme a etapa (Opus na especificação; trocar na
  implementação) e **vigiar o modelo ativo** para não estourar custo (lição do Fable).

## 👥 Time e pessoas

- **Decidido:** **tirar o Clayton do portal** imediatamente, para focar nas correções do V2 do
  Treasury Command. (Carlão nem registrou a entrada dele no portal hoje.)
- **Decidido:** estruturar como **dois produtos / dois times** — Treasury Command (novo) e
  Universe+portal (legado) — com a regra **"quem faz, cuida"**.
- **Decidido:** **Tayná vira dona do Atualiza**, com Clayton fazendo a transição. Primeiro um papo
  de **feedback** (como ela se vê na empresa); se fizer sentido, discutir **atualização da taxa
  horária** como sinal de investimento — condicionado ao desejo dela.
  - *Ajuste factual (Carlão):* a Tayná **não pediu aumento** ao Mota nem ao Carlão; sinalizou
    insatisfação com salário no início do ano (dúvida entre CLT/PJ). O papo de feedback do Andre
    **não** tocará em valores nesta primeira conversa.
- **Recomendado:** **Rodrigo e Júlia** cada vez mais próximos de produto/UX; Mota criará um
  **encontro semanal** dele com os dois. Rodrigo tende ao técnico, Júlia ao negócio — perfis
  complementares. Investir/treinar conforme motivação (os três — Clayton, Rodrigo, Júlia — estão
  bem motivados).
- **Recomendado:** manter o portal com o time atual (Alê/Su/Tayná tocam; Adolfo puxa novas
  funcionalidades). Avaliar **Tayná** (com o Carlão) assumindo partes do Atualiza.

## 🏗️ Produção, infra e suporte

- **Regra decidida:** **Andre e Clayton NÃO terão acesso à base do cliente** — só vão até o
  ambiente de desenvolvimento. Garante a passagem real de responsabilidade para a infra.
- **Decidido:** cliente em produção fica com **Fábio** e **Coutinho** (o time deles já teve ~4
  meses para se preparar; a responsabilidade é deles).
- **Recomendado (Mota):** estruturar, com o **Fábio** e usando IA, um **fluxo de suporte a
  produção**: triagem separando problema de **infra/máquina/rede** (Kubernetes) de problema de
  **produto**; abertura de chamados; **base de conhecimento** das resoluções; monitoramento
  (ex.: API falhou, saldo não atualizou). Não depender do bom senso/conhecimento individual.
  - **Compromisso do Andre:** conversar primeiro com o Fábio (ver o que ele já planeja) e pensar
    junto; possivelmente marcar reuniões específicas por assunto.
- **Recomendado (Mota):** envolver o Fábio na **construção** do novo modelo e deixar claro que o
  segundo semestre traz mudança profunda (o "Claude do passado não existe mais").
- **Ponto de atenção (Mota):** o Fábio precisa **delegar mais** ("virou parça da galera"); haverá
  "conversas difíceis" sobre pessoas do time dele (treinar, substituir ou contratar).

## 🧭 Filosofia orientadora (recomendações de fundo)

- **Processo acima de pessoas:** modelar o novo por processo, trazendo primeiro quem quer "comer
  grama"; reservar demanda de legado para quem ainda não está no ritmo do novo.
- **Desenvolver de dentro:** priorizar quem já tem **contexto** de negócio em vez de contratar
  especialistas de IA de fora. Só contratar quando Andre julgar necessário (talvez a partir do Q3;
  no Q2 o time "aprendeu muito e acabou estruturando").

---

Relacionado: [[xtpg]] · [[2026-07-03-resumo-aprofundado]] · [[2026-07-03-insights-takeaways]]
