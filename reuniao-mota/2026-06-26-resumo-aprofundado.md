---
title: "Conversa com Mota - Resumo Aprofundado - 2026-06-26"
date: 2026-06-26
tags:
  - projeto
  - arquitetura
  - lideranca
  - reuniao
  - status/rascunho
participantes:
  - Andre (Minoru)
  - Mota
---

# 🧭 Conversa com Mota — Resumo Aprofundado (2026-06-26)

> [!info] Contexto
> Check-in semanal (sexta-feira) entre Andre, Mota e o Carlão, com transcrição via Teams.
> A pauta central foi a demonstração, por Andre, do fluxo de desenvolvimento assistido por
> IA (Claude Code) aplicado ao [[xtpg]] — kanban de épicos, git worktrees, paralelização de
> agentes, geração de notas de reunião e fluxo de revisão de PR — além do alinhamento sobre
> o design das regras de alerta do cockpit e o planejamento da primeira entrega.

## 🗒️ Abertura informal — operadoras, "bolha" de IA e o IPO da xAI

A conversa começou com um papo descontraído sobre **troca de operadora de telecom**. Carlão
relatou a saga de migrar da Claro para a Vivo (problemas de instalação de equipamento, espera
pelo sinal do telefone fixo, queda de velocidade de 300 para 60–80 Mbps na Claro). Aproveitou
um plano mais barato na Vivo, subiu de 300 para 500 Mbps e reduziu a fatura. Andre contou seu
histórico de migrações sucessivas (Claro → outra operadora → Vivo → Claro), muitas delas por
preço, e ambos lamentaram a **má qualidade do cadastro das telecoms**: Andre chegou a cair no
Serasa por uma conta de telefone fixo que pediu para desativar e continuou sendo cobrada — um
problema agravado pela migração de empresas (Telefônica → BCP → Claro ao longo dos anos).

Carlão comentou que usou o **ChatGPT** para pesquisar qual operadora era melhor na sua região,
o que levou a uma reflexão maior: se todo mundo passa a pesquisar via IA em vez de acessar
sites diretamente, **como as fontes de informação serão alimentadas no futuro?** A discussão
evoluiu para a percepção de que os últimos lançamentos de LLMs estariam chegando a um "limite",
sem muito para onde crescer com os dados já existentes. Mota mencionou ter lido sobre a
reestruturação interna na empresa de Zuckerberg (Meta/Facebook/Instagram), onde engenheiros
de altíssimo gabarito estariam sendo realocados para **revisar e qualificar código gerado por
IA** — trabalhando totalmente monitorados pela própria IA, que aprenderia com o comportamento
deles. Houve polêmica e saída de talentos por causa disso.

Em seguida, falaram da **"bolha" de IA e do mercado financeiro**:

- Pressão de custos: Apple anunciando aumento de ~20% em iPhones e computadores por causa do
  encarecimento de chips/memória; Microsoft também sinalizando reajustes.
- A corrida das empresas de IA (OpenAI, Anthropic e outras) para **abrir capital** no segundo
  semestre, motivada pela necessidade de captar dinheiro enquanto o mercado financeiro permite.
- O caso emblemático do **IPO da empresa de IA do Elon Musk (xAI)**: abriu por volta de
  US$ 135 (preço institucional), começou a negociar a ~US$ 150, disparou até ~US$ 220 — fazendo
  Musk virar, segundo o relato, o "primeiro trilionário" — e depois **despencou de volta a ~US$ 150**,
  com mais de **US$ 400 bilhões evaporando** em cerca de 15 dias. Andre destacou que olhava os
  prospectos do Brasil e achava o valuation insustentável (mais de **200x o faturamento**) e
  estranhava uma empresa **sem fins lucrativos** atrair tanto capital.
- A promessa de **data centers espaciais** de Musk (sem problema de refrigeração) como a "nova
  briga" do setor — vista por ambos como muito distante e exagerada.

A conclusão compartilhada: o mercado está em **movimento de manada**, beirando a insanidade,
com exagero máximo — uma bolha. Mas também acreditam que sempre haverá **resistência** e que o
mercado vai se adaptar e estabilizar.

## 🤖 O coração da reunião — o fluxo de desenvolvimento com Claude Code

Andre avisou que estava **gravando e transcrevendo** a conversa (porque perde muita coisa,
principalmente "quando o Mota dá umas aulas") e compartilhou a tela para mostrar como está
conduzindo o desenvolvimento do produto com IA.

### 📋 Kanban, épicos e separação de responsabilidades (humano × IA)

- O projeto do Andre e do **Clayton** referencia, dentro do repositório, uma pasta com o
  **projeto/protótipo do Mota** anexado. A regra combinada: **ninguém grava nada nessa pasta** —
  ela é só leitura. Assim, quando o Claude gera documentos, já consegue apontar "vai na pasta do
  Mota, pega a especificação, pega o protótipo".
- A **fonte da verdade para a IA** são as **specs e os planos** (markdown). O **kanban visual**
  existe só para os humanos enxergarem em que ponto o projeto está — o Claude transfere as
  "caixinhas" para lá apenas com a descrição, sem "encher linguiça", porque os critérios de
  aceite e detalhes ficam dentro dos artefatos dele. Andre validou esse acordo diretamente com
  o Claude: "aqui é só pra humano ver; eu trabalho nas specs".
- O trabalho foi quebrado em **épicos (Q0–Q4)**. O Q3 já está praticamente resolvido e o Q2
  termina hoje. O Q0 também já saiu. Sobram cadastros e os investimentos (estes com o Clayton).
- Itens que dava para postergar ou dúvidas foram empilhados num **épico de débito técnico**
  (pequenos). A expectativa é **entregar essa primeira fase em ~2 semanas**.

### 🔀 Fluxo de Pull Request e revisão cruzada de contextos

Andre demonstrou o fluxo de PR com **dupla aprovação**, na verdade entre **dois contextos de
IA** (o dele e o do Clayton):

1. Ao fechar um item, Andre abre o PR descrevendo o que foi feito (ex.: **trocar o mock pelas
   APIs reais do Intraday**).
2. O contexto do Clayton gera uma **revisão**; conforme as pendências, marca "após ajuste" com
   observações.
3. Andre pega do lado dele: "PR #863, o Clayton colocou duas observações, veja se faz sentido."
   Se fizer, corrige; se não, **contesta e os dois conversam** — calibrando o contexto de cada
   lado.

Mota reforçou o princípio: não faz sentido revisar tudo no detalhe manualmente, senão "não faz
sentido ter IA". A IA tem que ser **extensora/potencializadora** — dar conta do que o humano não
dá conta — e o humano atua nos **pontos de checagem** para melhorar o contexto.

### ⚙️ Paralelização de agentes e git worktrees

- Andre quis adiantar o Q3 junto com o Q2 e pediu ao Claude para **paralelizar**. O Claude leu a
  especificação da máquina (memória, CPU) e sugeriu rodar até **4 processos em paralelo**, mas
  com **no máximo 2 sessões de Docker** (usado para subir banco de dados de teste).
- No começo, Andre montava manualmente **4 terminais** e ficava chaveando entre eles para
  responder às perguntas da IA. Depois descobriu um recurso de **"complete workflows"** que
  **orquestra as ondas automaticamente** (respeita dependências e hierarquia — sobe o quarto
  processo só quando o primeiro termina), mostrando uma "arvorezinha" do progresso. Mota comparou
  com a ferramenta que ele usa, mais manual, sem esse controle de ordem.
- Para rodar em paralelo de verdade, cada processo precisa de **branch e pasta separadas** — daí
  o uso de **git worktree** ("orquite", na transcrição). Andre contou que **descobriu o motivo do
  worktree sem querer**: rodando em paralelo, atualizou um arquivo numa branch, fez commit e
  removeu — e a outra sessão reclamou que o arquivo havia sumido. Aí "caiu a ficha": o worktree
  **isola fisicamente** cada trabalho em uma pasta/branch, evitando conflito. Funciona como uma
  branch, mas com diretório separado no disco.

### 🧠 Geração de notas de reunião a partir de transcrições

Andre explicou que, a partir de uma **aula do Elemar Junior** sobre **especificação na era da IA**,
montou um fluxo próprio. A tese do Elemar: antigamente você entrevistava o pessoal de negócio com
um caderninho anotando palavras-chave; **em 2026, se você não está transcrevendo/gravando, está
desatualizado**. O Elemar deu uma "receita" de prompts para transformar uma transcrição em
material útil — e Andre copiou as perguntas e montou sua própria automação (a skill **meeting-notes**).

Como funciona:

- **Passo 0/1:** pega a transcrição crua e identifica **termos e nomes que parecem equivocados**,
  devolvendo o entendimento da IA para o humano corrigir (ex.: confunde "Claude" com "Cláudio",
  "Júlio" com "Júlia"). A IA **guarda na memória** as correções recorrentes (que "Júlia é Júlia",
  que "Clódia é Cláudia"), de modo que nas próximas vezes já acerta.
- Depois, várias **saídas opcionais**: resumo abrangente, destaques/análise crítica, ideias
  principais, guia de estudos, frases impactantes, recomendações e compromissos (o "to-do" da
  reunião), frases "instagramáveis", **glossário** e **tabela ontológica** (que captura os **termos
  de negócio** ditos na conversa, com a explicação de cada um).
- Mota notou, impressionado, que o projeto já **associa automaticamente o protótipo do Mota** ao
  contexto, porque isso foi mencionado em conversas anteriores e ficou no aprendizado da IA. Tudo
  fica no repositório, acessível ao time.

Andre destacou o ganho: para o humano, revisar código linha a linha é inviável; o valor está em
**olhar por cima e atuar nos pontos de decisão**. E a ferramenta ainda **dá nota** à qualidade da
especificação — se a IA varre a transcrição de uma conversa de negócio e identifica pontos
obscuros, ela atribui uma nota (ex.: "nota 5/7, falta detalhar tal item") e devolve para a pessoa
de negócio aprofundar. **Adolfo** já foi gravar uma especificação nesse formato ("esquece Excel e
Word; é Markdown e transcrição").

## 💡 Decisão de design — regras de alerta no cockpit (segurança por empresa × regra global)

Na revisão do **cockpit** (tanto o de investimentos quanto o outro), surgiu um **ponto de decisão**
que Andre levou ao Mota:

> Quando o usuário cria uma **regra de alerta** (ex.: concentração, vencimento, % do CDI), ela deve
> ser definida **por empresa** ou de forma **global**? O usuário tem autorização/visualização **por
> empresa**, então faz sentido amarrar a regra a empresas específicas?

A resposta do Mota (que já tinha pensado nisso ao prototipar):

- **A regra é sempre global** — nunca por empresa. Definir regras por empresa aumentaria
  **complexidade gigantesca** e ficaria **inviável** conforme cresce o número de empresas; além
  disso, muitas regras (como nível de concentração) olham o **total**, por natureza.
- A **segurança por empresa é aplicada no momento da recuperação/exibição** dos dados: a regra
  monitora tudo, mas o que é mostrado ao usuário é **filtrado pelas empresas a que ele tem acesso**
  (via seletor de entidades). Exemplo dado: uma regra de "% abaixo de 75% do CDI" varre todas as
  posições, mas o usuário só vê as 20 posições / R$ 5 mi referentes às entidades autorizadas (ex.:
  filtrar por "regional" ou só "Bank of America").

Andre concordou: amarrar empresa na definição do alerta complicaria a vida do usuário ("e se eu
quero ver todas?"). **Conclusão consolidada:** regra de negócio **global**, **segurança por
empresa** aplicada como filtro na exibição. (Vale registrar como ADR — ver [[reuniao-mota/2026-06-26-recomendacoes-compromissos]].)

## 🚀 Planejamento — primeira entrega e primeiro cliente no XTPG

- **Entrega da primeira fase:** estimada em ~2 semanas. Andre vai dar **status no chat** entre
  segunda e quarta (não esperar até sexta para dizer se deu ou não deu).
- **Disponibilidade do Mota:** total — "qualquer momento, prioridade máxima, sem precisar marcar",
  para tirar dúvidas de negócio. Mota dispensou Andre e Clayton de irem ao escritório ("reclusão
  total", se preferirem).
- **Próximo passo após a primeira entrega:** colocar o **primeiro cliente no XTPG**, isolado, no
  ambiente **Oracle/Kubernetes** (o ambiente produtivo ideal), para ver **dados reais** e exercitar
  a dinâmica de **Open Finance** — justamente o ponto onde ainda há dúvidas. Mota destacou que
  trabalhar já no ambiente produtivo evita depender de mocks e dá mais profundidade/imersão,
  resultando numa **versão de mercado** com apenas ajustes finos depois.
- **Open Finance — bancos para teste:** a empresa já tem conta em **Itaú** e **Banco do Brasil**;
  o Mota pediu à **Vitória** para abrir conta em **Bradesco** e **Santander**, para ter os **4
  maiores bancos** e um teste realista. Falar com o **Fábio** para isolar o ambiente.

## ⚽ Encerramento — Copa do Mundo

A reunião fechou com papo de futebol (Copa do Mundo): o caos no trânsito de São Paulo em dia de
jogo do Brasil, o desempenho crescente da Seleção, polêmicas de arbitragem (VAR, o segundo gol do
Brasil, escanteios), e a avaliação de favoritos (França e Argentina jogando bem; o gol do Vinícius
Júnior no rádio). Avisos finais: na segunda há jogo do Brasil (Mota adiantou um compromisso para as
10h) e na sexta seguinte Mota terá a segunda microcirurgia no dentista — provavelmente ficará fora.

## ✅ Próximos passos (resumo)

- [ ] Andre: pedir ao Claude para **rever a pasta do protótipo do Mota** (houve refinamentos no
      cockpit de investimentos).
- [ ] Andre/Clayton: **entregar a primeira fase em ~2 semanas**; dar status no chat seg–qua.
- [ ] Andre: conversar com a **Alessandra** sobre o teste de exportação de PDF / conciliação.
- [ ] Implementar regra de alerta **global** com **filtro de segurança por empresa** na exibição.
- [ ] Após a entrega: subir o **primeiro cliente no XTPG** (Oracle/Kubernetes) com **dados reais
      de Open Finance**; falar com **Fábio** para isolar o ambiente.
- [ ] **Vitória**: abrir contas em **Bradesco** e **Santander** (já há Itaú e BB).
- [ ] Andre: buscar e enviar o **artigo do Elemar Junior** sobre transcrição/especificação.

> [!note] Notas relacionadas
> [[xtpg]] · [[cockpit]] · [[open-finance]] · [[reuniao-mota/2026-06-10-resumo-aprofundado]]
