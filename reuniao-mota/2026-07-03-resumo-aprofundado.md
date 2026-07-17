---
title: "Conversa com Mota e Carlão - Resumo Aprofundado - 2026-07-03"
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

# 🧭 Conversa com Mota e Carlão — Resumo Aprofundado (2026-07-03)

> [!info] Contexto
> Check-in semanal (sexta-feira, 03/07/2026). O arquivo reúne **duas reuniões 1:1 emendadas**:
> primeiro Andre com **Mota** (PO), depois Andre com **Carlão**. A pauta central foi o avanço
> do desenvolvimento do [[xtpg]] com Claude Code, o planejamento da primeira entrega do
> **Treasury Command**, o desenho dos ambientes (desenvolvimento × homologação), uma longa
> reflexão sobre o novo modo de trabalhar com IA, e a reestruturação do time (dois produtos,
> dois times) com movimentações de pessoas.

## 🚀 Parte 1 — Reunião com Mota

### 📊 Estado do desenvolvimento

Andre abriu relatando que o **Clayton** entrou "num circuito forte" há duas semanas e está
rendendo muito. A parte bancária foi priorizada e terminada, o que adiantou bastante o cronograma.
A equipe planejava, ainda naquela tarde, começar a publicar o ambiente para que na **segunda ou,
no máximo, terça-feira**, Mota já tivesse um ambiente para testar e navegar.

Algumas tarefas mais técnicas ficaram para fazer em paralelo — por exemplo, o cadastro/criação
de usuário (Claude já havia gerado até a tela). A ideia é criar os usuários manualmente nessa
primeira semana e ir ligando as integrações em paralelo.

### 🗂️ Fluxo de trabalho com Claude Code

Andre compartilhou a tela e demonstrou o fluxo que vem consolidando:

- **Kanban/painel de épicos:** conforme os épicos são resolvidos, eles somem do painel. Quem
  cria os cards de **débito técnico** é o próprio **Claude**, que separa os papéis: o escopo
  menor e mais rápido vai sendo feito, e o que é dívida técnica fica registrado para depois.
- **Fonte da verdade:** Claude usa o Azure DevOps apenas como reflexo — a fonte da verdade são
  os documentos de projeto (PRD e afins). O board serve à visualização humana, porque agora são
  duas ou mais pessoas olhando.
- **Especificação e plano de implementação:** ao mandar Claude analisar todas as US de dívida
  técnica sob um épico, ele gera a especificação e cria um **plano de implementação** em uma
  **pasta separada** (isolando documentação do código-fonte), com árvore de dependências.
- **Paralelização em waves:** Claude monta uma **árvore de dependências** entre tarefas, decide
  o melhor jeito de paralelizar e organiza em **ondas** (onda 2 depende da onda 1, etc.). Enquanto
  roda, é possível acompanhar visualmente quais atividades já foram feitas, quais faltam e quando
  passa para a próxima onda — pedindo aprovação no meio do caminho.
- **Revisão de PR automatizada:** quando Andre sobe uma alteração, Clayton precisa aprovar; antes
  disso ele aciona "Claude, avalia o PR #911". Claude faz a revisão, preenche observações no
  próprio PR, Andre ajusta e só então Clayton aprova. Isso evita que erros cheguem ao teste manual
  do Mota, **reduzindo o tempo de espera no "pingue-pongue"** e ganhando performance.

Restaram poucas tarefas (cerca de 12, de um total que já foi ~60), várias paralelizáveis, para
concluir rápido.

### 🌐 Ambientes e proteção de acesso

Andre propôs a estrutura de ambientes:

- **Desenvolvimento** — onde Andre, Clayton, Rodrigo e Júlia mexem.
- **Homologação** — onde **só o Mota** acessa, para não atrapalhar. O time testa no dev, sobe
  para homologação, e Mota valida ali com as **contas correntes reais da XTPG** (não mais os
  testes mockados).

> [!note] Decisão de custo
> Para evitar ~R$600/mês de um cluster novo, começam **reusando o mesmo cluster do portal**,
> apenas com ambiente separado. Se surgir necessidade de capacidade ou isolamento, a separação
> é rápida.

### 🔐 Dúvida técnica do Mota — armazenamento de credenciais Open Finance

Mota levantou uma dúvida sobre onde ficam guardadas as senhas/credenciais no fluxo de
consentimento: o usuário é redirecionado para a tela do banco/agregador, dá o consentimento,
e a partir daí é preciso uma chave interna para consumir a API liberada.

Andre explicou que existem o **client ID** e o **client secret** — que são da **XTPG** (cliente
da Pluggy) — e ficou de confirmar o detalhe de como cada cliente final é identificado sob o
"guarda-chuva" da XTPG (cada cliente isolado, com seu próprio consentimento/liberação de API).
Andre prometeu verificar e responder.

### 🤖 Reflexão longa — o novo modo de trabalhar com IA

Boa parte da reunião foi uma conversa conceitual, puxada por episódios concretos:

- **Fable e custo de tokens:** Andre contou que testou o **Fable** e "fritou o estoque" — por
  descuido deixou o modelo em Fable numa análise do Clayton e ele consumiu **70% da cota em meia
  hora**; Fable custa pelo menos o dobro do Opus. Aprendizado: escolher o modelo conforme a etapa
  (Opus entra com mais detalhe na especificação; troca-se na hora de implementar).
- **Aprendizado em tempo de desenvolvimento:** ao montar os "questionáriozinhos" e pedir a Claude
  para explicar problemas que ele mesmo não entendia (via *chat about this*), Andre aprende
  conceitos de infra/segurança **no momento em que mais precisa** — e por isso fixa melhor. Mota
  reforçou o paralelo com a academia (aprende-se muita coisa que "não tem nada a ver"), enquanto
  aqui o aprendizado é sob demanda.
- **Analogia engenheiro × matemático:** Andre comparou a IA gerando código ao papel do matemático
  (cria as teorias) e do engenheiro (descobre para que serve); a linguagem (C, Java) vira commodity
  — o valor está na aplicação, no produto final, e nas áreas de maior valor agregado (infra,
  segurança, arquitetura, negócio).
- **Mudança de paradigma:** Mota argumentou que não se trata de "emburrecer" com IA nem de
  simplesmente substituir peças, mas de **mudar de patamar** — mudar a maneira de organizar e
  trabalhar. É disruptivo: o R&D do futuro será totalmente diferente, com times humanos usando
  agentes especialistas (infra, código, gestão de projeto) integrados ao dia a dia.
- **Exemplos de agentes no dia a dia:** integração do Claude no **Slack** (agente por canal,
  virando especialista do conteúdo daquele canal) e o framework de agente autônomo **Hermes**
  (evolução de agentes anteriores) — instalado numa VPS, com arsenal enorme de ferramentas,
  operado por Telegram/Slack, 24/7, aprendendo o perfil do usuário. Mota destacou que o gargalo
  aqui é **confiança/segurança**: dar acessos amplos a uma máquina que "faz qualquer coisa" exige
  guardrails. Andre citou o **Elemar**, que criou uma "secretária" com IA (transcreve reuniões,
  gera resumos, prepara prospecção de clientes).

### 🌏 Geopolítica de IA — bloqueio do Fable e a Sakana AI

Mota contou o episódio (conforme o seu entendimento, a confirmar) do **governo americano
bloqueando o Fable**, e a reação do mercado: os japoneses lançaram, "de maneira relâmpago", a
**Sakana AI** — uma IA derivada que orquestra o que há de melhor (Opus, GPT-5.5 etc.), cara,
mas que **empatou com o Fable** nos benchmarks de avaliação de LLMs, com forte foco em
**identificação de vulnerabilidades/segurança**. Quando o Fable foi proibido, a Sakana virou
*top trend*; o mercado se adaptou tão rápido que os americanos acabaram **liberando o Fable de
novo** para não perder mercado. Moral compartilhada: não se deve pautar o negócio numa única LLM
que pode ser bloqueada "de um dia para o outro".

### 🔧 Detalhe técnico concreto — rate limiting e IP externo

No meio da conversa apareceu um caso real de aprendizado: ao expor uma **API externa** para o
webhook da Pluggy (que notifica eventos, como a criação de uma nova conta corrente), Claude propôs
proteções — **allowlist** de quem pode acessar e **rate limiting por bucket** (por IP externo),
para conter tentativas de sobrecarga/DoS. Claude ainda abriu uma US de dívida técnica antecipando
um problema: ao publicar na **Oracle Cloud (OCI)**, a API passará por um **balanceador de carga**,
e o rate limiting deixaria de ver o IP externo real (veria o do balanceador). A solução sugerida:
ler o IP carimbado pela Pluggy no **cabeçalho** da requisição. Andre destacou que, sem Claude, o
time nem teria antecipado esse problema — "não é só fazer mais rápido, é fazer um negócio melhor,
mais robusto".

## 👥 Parte 2 — Reestruturação do time (Mota e Carlão)

### 🧩 Dois produtos, dois times

Andre propôs enxergar o momento como **dois produtos**:

- **Treasury Command** (o novo, com Open Finance/cockpits).
- **Universe + portal** (legado, interdependentes, não dá para separar).

No segundo semestre haverá migração do Universe para o novo, mas por ora convivem. Andre defendeu
a regra **"quem faz, cuida"** (em vez de um time faz e outro dá manutenção), porque é na manutenção
que o time se acerta e aprende com os próprios bugs.

### 🔀 Movimentações de pessoas

- **Clayton sai do portal** imediatamente, para se concentrar nas correções do V2 do Treasury
  Command. Mota aprovou na hora.
- **Novo time já se formando:** Andre, Clayton, **Rodrigo** e **Júlia** (os três devs bem
  motivados). Rodrigo tende ao lado técnico; Júlia quer se envolver mais com negócio — se
  complementam. Mota quer trazê-los cada vez mais para o produto (papel de "gestores de produto"/UX),
  com encontro semanal específico entre ele e os dois.
- **Tayná assume o Atualiza:** como o Clayton segura o **Atualiza** (legado) e o portal já está
  tranquilo (time Alê/Su/Tayná toca, com Adolfo puxando novas funcionalidades), a proposta é a
  **Tayná virar dona do Atualiza**, com o Clayton fazendo a transição. Isso se encaixa com uma
  possível **atualização da taxa horária** dela (ela sinalizou insatisfação com salário no começo
  do ano, e teria pedido aumento direto ao RH — Carlão esclareceu que não foi feito a ele nem ao
  Mota). Andre conversará com Tayná e Carlão à tarde, com foco em **feedback** (como ela se vê na
  empresa), sem tocar em valores nesse primeiro papo.

### 🏗️ Fábio, infra e o novo modelo de suporte a produção

- **Cliente em produção** ficou com o **Fábio** e o **Coutinho** (não passou para todo mundo como
  Andre esperava). Regra importante: **Andre e Clayton não podem ter acesso à base do cliente** —
  vão só até o ambiente de desenvolvimento. Se tivessem acesso, "resolveriam" em vez de passar o
  bastão; o objetivo é a passagem real de responsabilidade.
- **Novo fluxo de suporte:** Mota quer que a área de infra (time do Fábio) faça a gestão dos
  contêineres e do monitoramento, e defende **não depender do bom senso/conhecimento individual**
  de cada pessoa. A ideia é usar IA para estruturar a triagem: separar problema de
  **infra/máquina/rede** (resolvível na administração de Kubernetes) do problema de **produto**
  (que precisa reproduzir ambiente e envolver o time de produto), com fluxo de chamados e base de
  conhecimento das resoluções. Andre concordou ("é aquela velha ideia de base de conhecimento") e
  ficou de pensar junto com o Fábio.
- **Fábio como liderança:** Mota vê o Fábio como o cara do futuro na área, cabeça aberta, mas
  observou que ele "virou parça da galera" e tem dificuldade de **delegar/liderar** — resolve tudo
  ele mesmo e acomoda o time. Há gente no time dele que talvez precise mudar; virão as "conversas
  difíceis" e será preciso "mexer nos vespeiros".

### 🧠 Filosofia de gestão — processo acima de pessoas, contexto como gargalo

- **Contexto é o verdadeiro desafio:** gerar código ficou fácil; o difícil é o contexto todo —
  negócio, empresa, sistemas legados, mercados atendidos e a atender. Andre relatou um caso ouvido
  na mentoria: uma empresa maior contratou "engenheiros de IA" e não deu certo, por serem novos e
  sem contexto de negócio/sistema. Conclusão compartilhada: melhor **desenvolver quem já tem o
  contexto** para usar IA do que trazer especialistas de fora — "o de fora vai falar mais do mesmo".
- **Processo mais que pessoas:** Mota defendeu pensar mais no processo do que em nomes específicos,
  adaptando ao perfil e ao momento de vida de cada um, e trazendo primeiro quem está disposto a
  "comer grama" nesse novo modelo. Quem não estiver no ritmo do novo terá demanda no legado
  (customizações, integrações). Esse primeiro grupo são os "desbravadores" que definirão o futuro
  da empresa.

## 🏖️ Encaminhamentos pessoais (Carlão)

Andre informou que tirará férias fracionadas neste mês: uma semana a partir do **dia 27** (levar
os pais para visitar o Matias) e outra semana em setembro/agosto (a confirmar), quando o filho
mais velho fará **cirurgia no ombro** (na verdade precisará operar os dois ombros e o joelho).
Por isso as férias foram quebradas — Mota está preocupado em fechar o módulo (versão para banco)
ainda este mês.

## 🔑 Síntese

O projeto entrou em ritmo de fechamento da primeira entrega, com o fluxo de desenvolvimento
assistido por IA já maduro (waves, revisão de PR, especificação automatizada). Em paralelo, a
liderança dá os primeiros passos de uma **reestruturação estrutural** — dois produtos/dois times,
"quem faz cuida", passagem de responsabilidade real para infra, e desenvolvimento das pessoas de
dentro (Tayná no Atualiza, Rodrigo/Júlia rumo a produto). O fio condutor de toda a conversa foi a
convicção de que o momento é de **mudança de paradigma**: não substituir peças com IA, mas mudar
o modo de trabalhar e organizar o R&D — com o contexto de negócio, e não a geração de código, como
o ativo escasso a proteger.

---

Relacionado: [[xtpg]] · [[2026-06-26-resumo-aprofundado]] · [[2026-06-19-resumo-aprofundado]]
