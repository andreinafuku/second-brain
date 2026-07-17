---
title: "Conversa com Mota e Carlão - Insights e Takeaways - 2026-07-03"
date: 2026-07-03
tags:
  - projeto
  - arquitetura
  - lideranca
  - reuniao
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Mota
  - Carlão
---

# 💡 Conversa com Mota e Carlão — Insights e Takeaways (2026-07-03)

> [!info] Contexto
> Compilado dos principais insights e aprendizados das duas reuniões 1:1 de 03/07/2026
> (Andre + Mota, depois Andre + Carlão) sobre o [[xtpg]], o fluxo com Claude Code e a
> reestruturação do time.

## 🤖 Fluxo de desenvolvimento com IA

- **A IA gerencia os próprios cards.** Quem cria os cards de dívida técnica no Azure DevOps é o
  próprio Claude; o board serve à visualização humana, mas a **fonte da verdade são os documentos
  de projeto** (PRD, specs), não o board.
- **Waves > fila manual.** Claude monta a árvore de dependências, decide como paralelizar e
  organiza o trabalho em **ondas**. Isso elimina o "espera terminar isso para ir para aquilo"
  feito na mão e dá visibilidade do que já foi feito e do que falta.
- **Revisão de PR pela IA encurta o pingue-pongue.** "Claude, avalia o PR #911" antecipa problemas
  antes do teste manual do Mota — ganha performance ao reduzir o tempo de espera entre quem faz e
  quem valida.
- **Escolher o modelo pela etapa, não por default.** Opus entra com mais detalhe na especificação;
  troca-se o modelo na hora de implementar. Deixar o modelo errado ligado tem custo real.

> [!warning] Custo de token é armadilha silenciosa
> O **Fable** consumiu **70% da cota em ~30 minutos** por um descuido (ficou ligado numa análise).
> Fable custa **pelo menos o dobro do Opus**. Verificar o modelo ativo em cada terminal/tarefa é
> parte da disciplina operacional.

## 🧠 Aprendizado e valor do trabalho

- **Aprende-se no momento em que mais se precisa — e por isso fixa.** Pedir a Claude para explicar
  o problema que ele mesmo levantou (via *chat about this*) transforma cada tarefa em aula sob
  demanda. É o oposto da academia, que ensina muita coisa "sem ter nada a ver".
- **Código virou commodity; o valor migrou.** Analogia engenheiro × matemático: não importa se é
  C ou Java — importa a aplicação e o produto final. O valor agregado está em infra, segurança,
  arquitetura e negócio.
- **Não é "fazer o mesmo mais rápido" — é fazer melhor.** O caso do rate limiting/IP no balanceador
  mostra que a IA antecipa problemas que o time nem consideraria, produzindo um sistema mais
  robusto, seguro e escalável.

## 🌐 Estratégia de IA e risco de fornecedor

- **Não pautar o negócio numa única LLM.** O episódio (a confirmar) do bloqueio do **Fable** pelo
  governo americano e a resposta relâmpago da **Sakana AI** (japonesa, orquestra Opus/GPT-5.5,
  focada em vulnerabilidades) mostra que depender de um único modelo é risco de "de um dia para o
  outro ficar na mão". Vale para modelo de IA e para agregador de dados.
- **Mesma lógica de arquitetura do XTPG.** Assim como não se amarra o sistema ao ID da Pluggy para
  poder trocar de agregador, não se deve amarrar o negócio a uma LLM.
- **Agentes no dia a dia são a próxima fronteira.** Claude no Slack (agente especialista por canal),
  frameworks autônomos como o **Hermes** (VPS + Telegram, 24/7, aprende o perfil do usuário) e a
  "secretária" do **Elemar** apontam o caminho — mas o gargalo é **confiança/segurança**: dar
  acesso amplo a uma máquina que "faz qualquer coisa" exige guardrails.

## 👥 Liderança e reestruturação de time

- **"Quem faz, cuida."** Separar quem constrói de quem mantém não funciona bem — é na manutenção
  que o time se acerta e aprende com os próprios bugs.
- **Dois produtos → dois times.** Treasury Command (novo) e Universe+portal (legado) convivem no
  segundo semestre; a estrutura de times reflete essa realidade até a convergência.
- **Passagem de responsabilidade tem que ser real.** Andre e Clayton **abrem mão do acesso à base
  do cliente** de propósito — se tivessem acesso, "resolveriam" e o bastão nunca passaria de fato.
- **Desenvolver de dentro > contratar de fora.** O gargalo é **contexto** (negócio, legado,
  mercados), não código. Contratar "engenheiro de IA" de fora tende a "falar mais do mesmo"; melhor
  investir em quem já tem contexto (Tayná no Atualiza, Rodrigo/Júlia rumo a produto).
- **Processo acima de pessoas.** Adaptar ao perfil e ao momento de vida de cada um, trazer primeiro
  quem quer "comer grama", e reservar demanda de legado para quem ainda não está no ritmo do novo.
- **Investir em quem está dentro.** Dar mais responsabilidade à Tayná (dona do Atualiza) e discutir
  atualização de taxa é sinal concreto de investimento — condicionado ao desejo dela.

> [!important] Não depender do bom senso individual
> O novo fluxo de suporte a produção deve ser **estruturado por processo e IA** (triagem
> infra × produto, base de conhecimento, chamados), não pela boa vontade ou conhecimento de um
> indivíduo. "Se depender do bom senso do humano, a gente sabe como é."

## 🗣️ Sobre delegação e conversas difíceis

- **Delegar é o desafio da liderança técnica.** O Fábio "virou parça da galera" e resolve tudo
  ele mesmo, acomodando o time — sinal de que precisa delegar mais.
- **Mexer nos vespeiros é parte do momento.** Reestruturar exige as "conversas difíceis" (treinar,
  substituir, contratar) que não podem ser adiadas indefinidamente numa ruptura de mercado.

---

Relacionado: [[xtpg]] · [[2026-07-03-resumo-aprofundado]] · [[2026-07-03-recomendacoes-compromissos]]
