---
title: "Conversa com Mota - Resumo Aprofundado - 2026-06-10"
date: 2026-06-10
tags:
  - projeto
  - arquitetura
  - lideranca
  - reuniao
  - status/revisado
participantes:
  - Andre (Minoru)
  - Mota
---

# 🧭 Conversa com Mota — Resumo Aprofundado (2026-06-10)

> [!info] Relação com a ata
> Versão analítica e aprofundada da [[2026-06-10|ata da conversa com Mota]]. A ata traz a estrutura objetiva (telas, decisões, encaminhamentos); aqui o foco é o **raciocínio, os trade-offs e a filosofia** por trás de cada ponto.

## Natureza e propósito da conversa

Call 1:1 com **compartilhamento de tela** entre **Andre (Minoru)** e o **Mota (Product Owner)**. Não é uma reunião de status: é um **alinhamento de visão de produto + filosofia de trabalho**. O Mota conduz, navegando pela especificação/protótipo que vem evoluindo, e o Andre valida, questiona pontos de fronteira (negócio × técnico) e traz de volta a perspectiva de implementação. O pano de fundo é o **novo sistema de tesouraria** que substituirá o [[referencia-universe|Universe]], com a meta declarada de **fechar a primeira versão até o fim de julho/2026**.

Há um subtexto importante: os dois estão se convencendo mutuamente de que a combinação **dado rico do [[open-finance|Open Finance]] + acabamento financeiro + IA** transforma o produto em algo vendável e diferenciado — e que a forma de construí-lo (AI-First) é tão decisiva quanto o que se constrói.

## 🎛️ Estratégia de produto e disciplina de escopo

O princípio que organiza tudo: **a spec é sempre maior que a entrega**. O Mota especifica "o melhor dos mundos" (deixa registrado o futuro), mas a entrega é o **mínimo vendável**, fatiada e priorizada. Isso aparece concretamente na decisão de marcar funcionalidades não prontas como **"em breve"** em vez de povoá-las com dados — discutido adiante.

O núcleo da v1 (Q2): **Login → Treasury Home → Open Finance → Bank Transactions → Investments → Intraday Position → Master Data (usuários/empresas/contas/hierarquia/moeda)**. Tudo o que não for essencial para esse ciclo funcionar (IA conversacional, e-mail de alertas, operações financeiras completas, multimoeda) é deliberadamente empurrado para depois.

A tese de valor que o Mota defende, e que o Andre "compra" durante a call: **só com os dados do Open Finance (saldo, transação, investimentos) já dá para entregar uma riqueza enorme** — uma "foto" dos investimentos sem sequer ter operações financeiras implementadas. O diferencial não é mostrar saldo e extrato (qualquer agregador faz), é **dar acabamento financeiro**: organizar a informação do jeito que o tesoureiro quer ver e agir.

## 🖥️ As telas, com o raciocínio por trás

### Treasury Home
Concebida para **começar quase vazia e crescer**. Estrutura prevista: bloco de **insights**, ponto de **interação com IA** (um link que aciona a inteligência — adiado), painel **Explore** (links que sempre remetem a uma tela real do sistema) e **[[today-at-a-glance|Today at a Glance]]** ("visão rápida"): os KPIs/cards de cada tela reunidos num lugar só, cada card remetendo à sua tela de origem com exatamente os mesmos registros. O Mota considera isso um diferencial ("se a gente já disponibilizar isso, já é diferencial").

### Open Finance
A "grande funcionalidade". Tela única de conexão: seleção de bancos, validação, **consentimento** (em regra, uma conta por consentimento; quando há mais, já vêm juntas). O Mota reconhece que a UX terá de se ajustar à realidade de cada banco — agência/conta, **código do operador**, **MFA** variável. Praticamente **sem CRUD**: quem mantém é o processo.

> [!note] Decisão de modelagem
> O Mota **desacoplou a criação da conta corrente da obrigatoriedade de ter uma empresa**. O raciocínio: a conta corrente é a informação que o Open Finance **sempre** entrega; já a relação empresa↔conta é incerta (ele não sabe ainda como virá). Então cria-se a conta sempre, e o vínculo com empresa fica como etapa posterior, sinalizada por um status de **triagem** para o usuário acompanhar o que falta validar/vincular. O princípio condutor é **"nunca parar o processo"**.

### Bank Transactions e Investments
O Open Finance alimenta as transações bancárias (extrato/movimentos) e os investimentos. Em **Investments**, cada linha é uma **operação financeira** e o detalhe abre as **parcelas** (Investment Transactions) — modelado a partir do que a API diz que oferece (ex.: debêntures). Sobre isso o Mota construiu o **Investments [[cockpit|Cockpit]]**: visão gerencial (total investido, liquidez imediata, vencimentos, rentabilidade média, prazo médio, concentração, agenda de vencimentos).

### Monitor de alertas e oportunidades (o "valor agregado")
Aqui está a virada de "mostrar dado" para "ajudar a decidir". Um **CRUD de regras** definidas pelo usuário, ancoradas na **política de gestão de investimento** da empresa, em três famílias:

- **Risco** — limite de concentração por banco/emissor (ex.: alertar acima de 35%; ao definir, o sistema faz uma **simulação imediata**: "com 10% de concentração você tem 31 posições afetadas, R$ 19 mi").
- **Risco operacional** — pendência prolongada, sincronização desatualizada.
- **Oportunidade** — piso de rentabilidade (ex.: operação abaixo de **95% do CDI** dispara alerta).

Notificação **no sistema** na v1; e-mail/outros canais ficam para o futuro. O Andre defendeu manter o alerta visível no sistema porque "pode gerar uma notificação".

### Intraday Position e Intraday Cockpit
Provavelmente a tela mais densa. Visão consolidada hierárquica **empresa → banco → conta corrente** (expande/recolhe), pensada para quem tem **milhares de contas** e não pode entrar conta a conta. Colunas: saldo inicial, entradas e saídas do dia, líquido do dia, saldo final, **disponível** (= total de investimentos) e **liquidez** (investimento + saldo). O clique abre o **detalhe do dia**: extrato do dia (filtros entradas/saídas/todas, Pix, tarifa) e os investimentos do dia, sem precisar ir para outra tela; histórico completo fica em Bank Transactions.

> [!important] Diferencial — os carimbos de status
> Materializam a filosofia do Mota de **"não prometer mundo perfeito, mas tornar o imperfeito visível num lugar só"**: **atrasado** (última leitura do Open Finance anterior ao horário atual, com timestamp), **sem abertura** (não compôs o saldo inicial), **negativo**, **parcial** (conta que o usuário ainda quer acompanhar, "não pode sair do radar") e **fechado** (decisão tomada, sai do radar do dia). O argumento de venda: "o problema não é ter problema, é o cara não conseguir identificar **onde** está o problema" — então o sistema "explode" tudo num único painel.

O **próximo passo** dessa tela é sair do **diagnóstico** para a **ação**: fazer o **[[cash-pooling]]** (mover saldo para cobrir contas negativas, evitando juros). O **Intraday Cockpit** consolida KPIs (total por banco/empresa, saldo por faixa de liquidez) e aplica o mesmo motor de regras, agora com indicadores como **queima de caixa** (saída líquida do dia) e **descoberto** (saldo < 0).

### Master Data — Usuários e Segurança
O foco do Mota nessa parte foi **acesso e segurança**, que ele identifica como "o maior problema que a gente pode ter" (mais que funcionalidade, já que a v1 tem essencialmente uma funcionalidade só). Papéis herdados do sistema atual: **administrador**, **tesoureiro** (operacional/supervisor) e **visualizador** (consulta).

Mecanismo central: **concessão de acesso por árvore de hierarquia** — o admin escolhe o **nó** e tudo abaixo herda, com validade opcional. Duas mudanças de governança importantes:

- **Toda concessão passa pelo Admin** — acaba o antigo papel do supervisor que distribuía acesso. O Andre conectou isso a como **ERPs** fazem (o correto é o admin conceder, não dar amplos poderes ao pessoal de operação) e a clientes com **auditoria forte** sobre privilégios financeiros (a pagar, a receber, tesouraria).
- **[[maker-checker|Maker-checker]]** (dupla autorização): um cria, outro aprova, antes de o acesso valer.

Um insight de design que o Andre destacou: o Mota **reaproveitou a mesma "árvore" de hierarquia em todas as telas e relatórios**, o que **eliminou o painel de parâmetros** do sistema antigo — mesma facilidade de seleção em todo lugar.

> [!important] Decisão em aberto — hierarquia de segurança
> Editar uma hierarquia já usada em segurança é arriscado (pode quebrar relatórios e acessos). Duas opções: (1) bloquear a edição de hierarquias vinculadas à segurança até desfazer as concessões; ou (2) criar um **tipo de hierarquia específico para segurança**, sob responsabilidade do Admin, deixando as hierarquias de negócio (legal, geográfica) livres. A inclinação foi pela **opção 2** — e o Andre sugeriu validar com o Claude/Codex.

### Moeda
Conversão (dólar, euro) vive no Master Data, mas a **v1 será só BRL** — é mercado brasileiro e adiar a conversão de taxas não tem impacto relevante na primeira versão.

## 🤖 Metodologia AI-First — a parte mais "filosófica"

Essa foi metade da conversa e é onde os dois mais se animam. A tese, reforçada por **duas fontes externas que coincidiram na mesma semana** — o **curso do [[referencia-aulas-elemar-masterclasses|Elemar]]** (na quarta) e um **podcast do Paulo Silveira** (fundador da **Alura**) com RHs de **Itaú** e **iFood** — é uma **mudança de mentalidade** no uso de IA de codificação:

- **Antes** (começo do ano): tratava-se o Claude como **dev júnior** — você desenha a solução e manda ele fazer o "trabalho bruto".
- **Agora**: trate-o como o **melhor dev sênior do mundo**. Você não chega com a solução pronta; você passa **problema + contexto + critérios de aceite** e deixa a IA **propor opções**, depois **debate** as opções. Dar a solução pronta **limita** a IA ao seu próprio conhecimento ("cada um tem o Claude que merece"; "às vezes seu consumo é mais limitado que o dele").
- A analogia do Elemar (caso do **Google Chrome** e a lista de URLs maliciosas de ~10 GB): você não diz onde guardar os dados; você dá o problema e os **critérios** (performático, barato, sem ponto único de falha) e deixa a engenharia emergir. O Itaú relatou a mesma armadilha no podcast: ao ditar a solução, "estavam limitando o Claude a soluções que já usavam no passado".

Isso aterrissou em **exemplos concretos**:

- **No código (caso de ontem):** o problema das **linhas com valor zero** no fluxo de caixa (há até um flag disso no Universe; o volume de zeros supera o de não-zeros). Em vez de ditar, descreveram o problema → o Claude deu **3 opções** → debateram, escolheram a melhor → resolvido **em uma linha**. (Contexto: o **Adolfo** abrira mão de exibir linhas zeradas; a **Suely/Su** estava implementando e já tinha uma ideia, mas o Andre insistiu em deixar a IA propor.)
- **Na spec (Mota):** ao pedir "tesouraria, mas **AI-First**, com agentes", o Claude sugeriu prever **sponsor de negócio** e **dono técnico** para cada agente de IA antes de conceder autenticação — algo que o Mota admite que **não pensaria sozinho**. Daí o lema dele: "**vou dando linha**, não tiro nada daqui".
- **Dar linha na prática (fetch × Axios):** o **Rodrigo** sugeriu **Axios** para chamadas de API no React; em vez de impor, o Andre deixou o Rodrigo **questionar** o Claude, que apresentou opções e **convenceu** que o **fetch** era mais leve. O Andre foi pelo fetch — decisão da IA argumentada, não imposta.

### Workflow operacional do Andre
- Definiu o escopo do **Q2** para o Claude e fixou a **fonte da verdade = front-end do protótipo** (não reimplementar; **trocar mock por API**).
- O Claude gera as **US** (cards no **Azure DevOps**) com descrição + **critérios de aceite**, e a partir dos critérios gera **testes automatizados**. Os casos custosos (ex.: **P95**) ficam para rodar **na mão**: o Andre roda, cola o resultado, o Claude valida e aprova a US (move para resolvida) e parte para o plano.
- Está envolvendo **Júlia** e **Rodrigo** (acompanhando o protótipo) e o **Clayton** (no código) para que cheguem preparados quando entrarem as novas funcionalidades.

### Workflow do Mota
Descreve um dia já **agent-first**: começa perguntando aos agentes "onde paramos ontem?"; usa agentes para agenda e Google (não pesquisa mais "na mão" — manda o agente trazer e salvar no diretório); abre o **Codex** por cima do mesmo diretório no **VS Code** para achar furos; usa **MCP** para manter **contexto atrelado** ("se você sai do fluxo, perde contexto e duplica trabalho"). A leitura comum dos dois: a **nova geração** que está se formando já virá com essa fluidez, e os gestores terão de saber trabalhar com ela.

## 📣 Tração comercial e posicionamento

O Mota já **apresentou as telas a 2 clientes** e o feedback foi excelente — "**melhor do que o mercado de Open Finance está mostrando**", **sem nem falar de IA**, só a navegação e a riqueza dos dados. Um quer ser o primeiro: **Rede D'Or** (~**2 mil contas**, muitos bancos), que é **on-premise** e **ainda não usa o produto da XTPG** — entra, portanto, como **módulo totalmente novo**, com cadastro inicial de usuários/hierarquia/empresas feito "na mão" e depois o cliente loga, dá consentimento e opera. Posicionamento estratégico: o produto **não depende da base instalada** — vende também para **prospects**. O Mota é honesto sobre o risco: "tudo isso é hipotético, espero que o Open Finance entregue; depois a gente vai ver as imperfeições" — e a resposta a isso é justamente tornar as falhas visíveis (os carimbos) e complementar os dados onde o Open Finance não cobre.

## 🧑‍💼 Ângulo de liderança / pessoal

Boa parte do fim da conversa é o Mota **insistindo para o Andre se desconectar do portal** por ~um mês e mergulhar no novo sistema ("nas próximas duas semanas não vou nem atender; volte depois"). O Andre concorda e vai **formalizar**: já vem **delegando** (chamou **Alê/Alessandra** para o fluxo de caixa, vai avisar o **Carlão**, e o **Adolfo**, que centraliza demandas, acionará alguém na ausência dele). Há uma reflexão sobre disponibilidade: "não dá para estar totalmente disponível — tem que dar uma canseira no outro lado para pensarem". E uma observação cultural que o Andre traz do podcast: a nova geração já não "vê e-mail" — integra tudo a Teams/ferramentas, e **reuniões viram transcrições/atas** (ele mesmo diz que pretende não entrar em certas reuniões e só pedir ao **Clayton** o resumo da ata).

## 🎯 Síntese / próximos passos

- **Meta:** fechar o pacote v1 (Open Finance + Bank Transactions + Investments/Cockpits + Intraday + Master Data/Segurança) **até o fim de julho**, com atrativo de marketing para base **e** prospects.
- **Produto:** confirmar a **hierarquia de segurança separada** (opção 2); próxima spec do Mota é a **edição/complemento** de dados que faltarem no Open Finance.
- **Método:** consolidar o modo AI-First (problema + critérios, não solução), com a IA propondo e o time debatendo.
- **Pessoas:** Andre se afasta do portal ~1 mês (delegação a Alê/Alessandra, Carlão, Adolfo) para focar no novo sistema com o Mota.

## 🔗 Notas relacionadas
- [[2026-06-10]] — ata objetiva desta conversa
- [[2026-06-10-analise-critica]] — análise crítica dos momentos mais interessantes
- [[2026-06-10-insights-takeaways]] — compilado de insights e takeaways
- [[open-finance]] · [[referencia-pluggy]] · [[referencia-universe]]
- [[cockpit]] · [[cash-pooling]] · [[maker-checker]] · [[today-at-a-glance]]
- [[ddd-domain-driven-design]] · [[load-bearing]] · [[referencia-aulas-elemar-masterclasses]]
