---
title: "1:1 com Tayná - Resumo Aprofundado - 2026-07-03"
date: 2026-07-03
tags:
  - lideranca
  - projeto
  - reuniao
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Tayná
---

# 🧭 1:1 com Tayná — Resumo Aprofundado (2026-07-03)

> [!info] Contexto
> Conversa 1:1 (sexta-feira, 03/07/2026) entre Andre e **Tayná**, dev do time do [[xtpg]]. A
> pauta central foi o convite para a Tayná assumir novas responsabilidades — **dona do portal** e
> **dona do Atualiza** — no contexto da reestruturação em dois produtos/dois times. A conversa
> incluiu feedback sobre o dia a dia dela, o entendimento do módulo Atualiza e uma longa
> demonstração do fluxo de transcrição de reuniões + a sequência de prompts do Elemar.

## 🗒️ Abertura — bastidores e alinhamento com o Mota

Antes da chegada da Tayná (que teve consulta e atrasou), Andre relatou o alinhamento já feito com
o **Mota** naquela manhã:

- O foco do Mota em julho é a **versão 2** (Treasury Command); por isso a ideia de **tirar o
  Clayton do portal** para acelerar.
- Andre "plantou a semente" da estrutura de **dois produtos** — Universe+portal (legado) e o novo
  (Treasury Command) — e o Mota concordou 100%.
- Sobre quem fica no produto novo: Andre, Clayton e a dúvida sobre Júlia e Rodrigo. O Mota sugeriu
  que os dois fiquem **com o pé nos dois produtos**, já que a demanda do portal é menor e "está
  mais arredondada" (eles não têm a responsabilidade de "ver o todo" que Andre e Tayná tinham).
- O Mota pediu para **incluir o Fábio** quando começarem a pensar no **monitoramento** do sistema
  do Treasury (a primeira entrega depende de monitorar serviços; o time gera logs e depois alinha
  com o Fábio o que mais oferecer para o monitoramento dele).
- Sobre o **Atualiza**: o Mota topou a ideia de a **Tayná assumir**, condicionado a ela querer.

## 💬 Feedback — o dia a dia da Tayná

Andre abriu espaço para saber como ela está. Pontos que ela trouxe:

- **Processo praticamente o mesmo**, mas agora mexe mais com **ambientes** (sobe no ar quando Júlia
  e Rodrigo pedem), vê mais a parte de **Kubernetes** (com apoio do Claude) e segue resolvendo
  **bugs**.
- O **fluxo de caixa** que vinham desenvolvendo está **quase 100% / bem estável** — aparece um bug
  ou outro, mas está maduro. Há um backlog que o **Adolfo** vai começar a puxar.
- **Visão de ponta a ponta:** ela valoriza entender **todo o processo/ciclo**, não só o código —
  "com IA, o código virou commodity; o mais importante é saber realmente todo o processo". Já faz
  **primeiro diagnóstico** olhando front-end, back-end e banco de dados (ex.: variação cambial
  errada → confere de onde vem o erro antes de ir à solução).
- **Uso do Claude:** entrou mais para configurar (junto com o Clayton), fez investigações, mas não
  se aprofundou. No **Docker/deploy** se vira bem (scripts e documentação prontos). Está
  **aprendendo mais com o Claude** do que com tutoriais na internet — pergunta, vê as opções junto,
  estuda com a orientação dele; destacou a "humildade de perguntar" e como isso **acelera** o
  aprendizado, ainda mais por ele ter todo o **contexto do projeto**.
- **Configuração da IA nova:** mexeu na **compactação/sumarização de contexto** e na
  **identificação dos agentes** (hoje há o **geral** e o **cash**; o agente de **mercado** ainda
  não está pronto).

## 🏗️ Proposta 1 — assumir o portal

Andre apresentou o desenho dos dois produtos e explicou que o **Clayton vai deixar o portal aos
poucos** para focar nos **testes do Treasury** (que virão em volume). A ideia é a **Tayná assumir
o portal**, aproveitando sua visão do todo (front-end, back-end, banco). No dia a dia ela toca
sozinha, mas pode chamar Andre e Clayton para decisões de arquitetura ou segunda opinião — "a
gente conversa em três".

> [!note] Filosofia "quem faz, cuida"
> Andre reforçou que não é "fazer na frente e alguém corrigir a cagada atrás" — cada time toca o
> seu produto: o novo com Andre/Clayton, o portal com a Tayná.

## 🔄 Proposta 2 — dona do Atualiza

Andre explicou o **Atualiza** em detalhe:

- É o módulo que mantém, na base do sistema, o **cadastro de cotações e índices** — dólar, euro,
  taxas econômicas como o **IGP-M**.
- Atualiza essa base fazendo **web scraping** de sites ou lendo **APIs**, dependendo da fonte.
- Rodam **jobs schedulados** (ex.: diariamente busca as cotações).
- Novas fontes surgem constantemente; quem faz o levantamento (entende o site, o que buscar) é
  geralmente a **Daiane**, e o time cria o "motorzinho de busca" (scraper/leitor de API) por fonte.
- **Stack:** está em **.NET** (não Python).

> [!example] Por que "Atualiza"
> A Tayná deduziu o nome: o sistema precisa **atualizar sempre** os índices. Andre ilustrou com o
> caso de uma compra em dólar cuja fatura só vence no mês seguinte — a cotação usada é a da **data
> de pagamento**, não a da compra, então as taxas precisam estar sempre atualizadas.

A Tayná se interessou, já pensando na parte técnica (perguntou se cria do zero ou se já existe algo
— Andre confirmou que **já existe** e pega de várias fontes). Como dona, ela avaliaria prós/contras
de **subir a versão** do Atualiza, sempre decidindo em conjunto e com apoio do Claude na pesquisa.
Ela resumiu bem o convite: **"dar mais responsabilidade, quase nível sênior"**.

## 🎥 Demonstração — transcrição de reuniões e prompts do Elemar

Andre mostrou o fluxo que quer que a Tayná adote ao conversar com pessoas de negócio (Daiane,
Adolfo, Fernanda) e ao fazer a passagem com o Clayton:

- **Gravar e transcrever** (avisando antes) em vez de anotar — "tem muito termo e conhecimento que
  se perde se não gravar". Ferramentas testadas: **Granola** (trial 30 dias) e **MS Teams**;
  também dá para transcrever via Python.
- **Sequência de prompts do Elemar** (link a compartilhar): o **primeiro prompt** faz o Claude ler
  toda a transcrição e **apontar termos/nomes que parecem equivocados** para o usuário corrigir,
  **antes** de gerar qualquer resumo — evitando resumos que "ninguém falou isso". Andre mostrou na
  prática, usando a própria transcrição da conversa com o Mota (o Claude acertou André Minoru,
  Clayton como humano, Claude vs Clayton, Carlão, e corrigiu "Rodolfo → Adolfo").
- Depois vêm os demais: resumo completo/abrangente, destaques, guia de estudos (pergunta/resposta),
  frases impactantes, recomendações e compromissos, frases "instagramáveis", **glossário** (útil
  ao conversar com negócio — o Claude complementa termos financeiros consultando a internet) e a
  **tabela ontológica** (liga um termo a outro para entender o **contexto**).

## 🗓️ Encaminhamentos

- Andre já falou com o **Carlão**, que deve **chamar a Tayná na segunda-feira** para formalizar as
  novas responsabilidades e provavelmente tratar da **revisão da taxa horária** dela.
- O **Clayton** fará a **passagem** do Atualiza aos poucos; se preciso, conversas iniciais junto
  com a **Daiane**.
- A Tayná topou e achou o fluxo de transcrição "muito útil"; como não conhece o Atualiza, disse que
  provavelmente reverá a gravação da passagem.

## 🔑 Síntese

O 1:1 formalizou, na prática, o movimento de **desenvolver de dentro**: a Tayná — que já tem visão
de ponta a ponta e usa o Claude com desenvoltura — foi convidada a assumir a **dona-ção** do portal
e do Atualiza, um salto "quase sênior" em responsabilidade, com apoio próximo de Andre e Clayton e
transição gradual. Em paralelo, Andre transferiu conhecimento de **método** (gravar/transcrever +
prompts do Elemar), reforçando a tese recorrente do time: com a IA cuidando do código, o valor está
em **dominar o processo, o contexto e o negócio**.

---

Relacionado: [[xtpg]] · [[2026-07-03-resumo-aprofundado]] · [[2026-07-03-recomendacoes-compromissos]]
