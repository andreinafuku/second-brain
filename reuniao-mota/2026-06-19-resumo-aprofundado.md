---
title: "Weekly R&D com Mota - Resumo Aprofundado - 2026-06-19"
date: 2026-06-19
tags:
  - projeto
  - arquitetura
  - lideranca
  - reuniao
  - status/revisado
participantes:
  - Andre (Minoru)
  - Mota
  - Carlão
---

# 🧭 Weekly R&D com Mota — Resumo Aprofundado (2026-06-19)

> [!info] Natureza da conversa
> Weekly de R&D, com participação inicial do **Carlão** (status do portal/Universe atual) e, no grosso, um **1:1 entre Andre (Minoru) e o Mota (Product Owner)**. O tema central é o **novo sistema de tesouraria** que substituirá o [[referencia-universe|Universe]]: como fatiar a primeira entrega para **validar o Open Finance na prática**, conciliar isso com as férias do Andre e aprofundar a filosofia **AI-First** e as dúvidas de modelagem técnica.

## 🗂️ Bloco 1 — Status do Carlão (portal atual)

Semana mais tranquila, com o analista tocando a maior parte. Pontos:

- **Autenticação movida do usuário para o *service*** (mudança previamente combinada), em teste durante a semana.
- **Bug de exportação de PDF corrigido** e já **em produção**.
- Acompanhando o que o **Renato** fez na **criação de clientes**; o ambiente novo de desenvolvimento ainda não tem usuário, então não dá para autenticar/usar API ali (fica para depois).
- **Produto novo** em fase final de testes com o **Adolfo**; apresentação a cliente hoje, com entrada de novo ambiente provável na semana seguinte (cliente **Neo / Neoenergia**).
- Começando a desenhar **fluxo padrão de atualização de ambientes** (clientes já testam em homologação e migram para o mesmo ambiente).
- **Ambiente comercial do Ciro** atualizado para a versão mais recente, para as próximas apresentações usarem o sistema atual. O conhecimento de atualização está sendo **multiplicado** (Adolfo → Fernanda → Ciro/comercial).
- Mencionada uma alteração de segurança de 2024 (proibir entrada em telas sem autorização explícita) que teve "probleminha" — a conexão do Carlão caiu antes de detalhar.

## 🤖 AI-First na prática — o que apareceu de concreto

### Git worktree: "senti na pele para que serve"
O Andre rodou duas sessões do [[referencia-aulas-elemar-masterclasses|Claude]] em paralelo no mesmo diretório e bateu o problema clássico: uma sessão criou um documento, fez o PR e o **merge**; a outra sessão acusou que "tinha um arquivo aqui que sumiu". Era exatamente o documento criado na outra branch. O Claude explicou: para rodar duas sessões paralelas na mesma máquina **sem conflito**, use **git worktree** — uma working tree separada por sessão. Lição registrada: paralelizar dentro da mesma branch gera colisão; **worktree isola**.

### "Dar linha" — quanto mais você comanda, menos a IA propõe
O Mota observa que, quando **comanda pouco**, a IA o **norteia** para a escolha técnica mais adequada (foi assim que ele conheceu o worktree); quando **comanda demais** (ex.: "quero uma worktree"), a IA apenas executa. A síntese dos dois:

> [!quote] A linha tênue do AI-First
> "Se você mandar, ele faz. Se mandar pouco, ele recomenda. Se mandar errado, ele te corrige." — o ideal é **dar o problema e os critérios** e deixar a engenharia emergir, em vez de ditar a solução.

### Mock × API com *feature flag* e padrão *adapter*
No planejamento de sexta/segunda com o **Clayton**, o Claude estruturou a troca progressiva de **mock por chamada de API**. Mecanismo:

- Uma **variável no `.env`** (`true`/`false`) decide se a tela consome **API real** ou **mock**. Conceito que "o pessoal usa lá fora" e a equipe não conhecia.
- Um **adaptador (interface)** pluga, de forma **transparente**, as funções do protótipo (quando mock) ou as chamadas reais (quando API). Júlia/Rodrigo podem continuar no mock do protótipo enquanto a API evolui.
- **Fonte da verdade = front-end do protótipo**: não se reimplementa, **troca-se o mock pela API**.

## 🎯 Planejamento do Q2 — os quatro épicos

O Claude gerou o plano em **visão macro** ("nível da água"), aprofundando só quando se entra para resolver:

1. **Análise de impacto** — comparar o que já existe no protótipo com as **coisas novas** que o Mota especificou; ajustar o que conflita.
2. **Open Finance** — amarrações finais das chamadas de API.
3. **Master Data + Identificação** — cadastros mínimos e mapeamento de identidade.
4. **Investimentos**.

Regra de trabalho: **quem terminar puxa o próximo**, dividindo tarefas para não colidir.

> [!note] O ponto de fricção da semana
> Ao pedir "**necessário e suficiente** para o intraday/análise de investimento funcionar", o Claude **supôs** que haveria uma carga inicial de empresa e conta corrente e planejou só **consulta**. Mas é preciso o **mapeamento da conta corrente do provedor ([[referencia-pluggy|Pluggy]]) contra a conta cadastrada**, para saber a quem pertence o saldo que chega — e isso exige um mínimo de cadastro (telinha de CRUD de empresa/conta).

## 🪶 A decisão central — implementar o mínimo (YAGNI)

O Andre chegou à reunião com a dúvida: cadastro de empresa **completo** (com status, travamentos, datação) ou **mínimo**? O Claude ofereceu **três níveis**:

| Nível | O que entrega |
|---|---|
| **Subset alinhado ao protótipo** | Campos do protótipo (identidade legal, condicional, moeda) + **hierarquias**; rápido e melhor custo |
| **Subset rico (intermediário)** | Todos os campos escalares + status de estado simples; difere só em subsistemas caros |
| **Completão** | Tudo o que está especificado (estados, travamento, datação) — demora mais |

> [!important] Decisão: vai no mínimo
> O norte é **a informação que vem do Open Finance**. O mínimo para rodar:
> - **Empresa** (entidade obrigatória) e suas **hierarquias** (agrupamento que alimenta o **filtro presente em todas as telas** — peça-chave do [[cockpit|Cockpit]]).
> - **Conta corrente** abaixo da empresa — mas **não vinculada à empresa de início**, porque o Open Finance "não olha empresa/hierarquia, ele entrega a conta e os dados"; o vínculo vem **depois**.
> - **Banco** como derivada da conta (só para carimbar).
> - **Moeda**: só lista, movimentos **apenas em BRL** — sem manutenção de dólar/euro agora.
> - **Calendário, método de pagamento, contraparte, reference data**: não funcionais nesta versão.
> - **Usuário**: o mínimo para logar e definir acessos por hierarquia.
>
> O princípio é o **YAGNI** — "você não vai precisar disso agora": nunca adicionar funcionalidade ou otimização até que seja **estrita e imediatamente necessária**.

O Mota reforça que **ele próprio continua enxugando** o protótipo, então a versão que o Andre está construindo "vai mudar pouco" — está sendo simplificada em paralelo. O CRUD mais elaborado que vale manter é o de **regras de alerta** do Cockpit, mas isso depende de entender como o dado virá do Open Finance.

## 📅 Férias × entrega de julho

- O Andre sai de **férias a partir de 20/julho**; planeja **dividir em 7 + 7 dias** (uma semana agora, a outra metade em agosto).
- Combinou: **terminar a entrega até ~17/julho**, sair na semana do dia 20 e **voltar em 27/julho**. Júlia, Rodrigo e Clayton "sobrevivem" essa semana.
- O Mota quer uma **versão funcional antes** das férias, inclusive apresentável/instalável (ex.: **Rede D'Or**). Visão otimista: **algo para ver já na semana do dia 3** ("versão de experimentação").
- Decisão final do Andre: ficou **propenso a sair só em 27/julho** ("quanto antes entregar, melhor"). O Mota: "você que manda".

## 🚀 Estratégia de entrega — versão de experimentação primeiro

> [!tip] "Deixa ele fazer, depois a gente ajusta"
> A tática combinada: ir com **tudo na primeira versão acabada**, mesmo na dúvida — "deixa ele fazer, a gente pega e ajusta". Ganha-se **performance** e descobre-se o produto final; depois refina-se ("isso aqui não ficou bom, arruma"). O objetivo da versão do dia 3 é **experimentação**: o Mota coloca **dados reais**, conecta a própria conta e observa **como a API do Open Finance se comporta** (o que vem, como atualiza).

### Liberação de recursos
- **Clayton liberado para intensidade máxima** (inclusive fim de semana, se tiver disponibilidade) — "mês focado, só emergência absoluta".
- **Assinaturas do Claude**: "já vira a chave agora e pede reembolso"; mandar os dois (Andre e Clayton).
- **Não economizar créditos** neste momento: "não fica economizando, mete o pau; se estourar, a gente vê e arruma usuário/plano". Prioridade é ter a versão experimental **o mais rápido possível**.

## 🧠 Filosofia — conhecimento, commoditização e diferencial

A aula de quarta (Elemar) abriu com a escada do conhecimento: **dado → informação → conhecimento → sabedoria**. Cuidado para **não parar na informação** — é preciso **aplicar** para virar sabedoria/prática.

> [!quote] O que diferencia na era da IA
> "Se você consegue fazer rápido, muita gente consegue fazer rápido igual. A diferença é o **conhecimento e a sabedoria** que você adiciona ao sistema." — daí o Andre conectar ao **Cockpit**: a entrega não é "fazer o sistema atual mais rápido com roupa nova", é **adicionar conhecimento de tesouraria** (acabamento financeiro, operações).

O Mota amplia: o mercado de software **explodiu** ("todo mundo hospeda e tem o melhor software"), gerando burburinho até na base instalada. Mas o diferencial está na **commoditização** — qualquer um faz a primeira tela bonita, "bonitinho mas ordinário"; poucos resolvem a **complexidade real** da tesouraria (governança, multimoeda/câmbio, lógica de alertas, estruturas multinacionais). A estratégia é **especializar-se e acumular complexidade** (futuro: integração com **SAP**, cotações de mercado), tornando o modelo **difícil de replicar** — e, mesmo replicado, virar uma "fábrica de software" cara de manter.

> [!note] Posicionamento modular
> O Open Finance pode ser vendido como **módulo independente**, inclusive para quem **não é cliente** da base instalada — abre outro patamar comercial. Feedbacks de mercado já vieram: clientes disseram que o que viram "é melhor do que tudo até hoje".

## 🧑‍💼 Evolução dos papéis — Júlia e Rodrigo

> [!important] "O design de tela puro acabou"
> Com a IA fazendo o desenho, o **design puro** perde função. Júlia e Rodrigo precisam **se aproximar do negócio** (entender as dores do tesoureiro) e migrar para um papel de **produto + experiência (UX) profunda** — usar a IA "na veia", interagir, jogar telas fora quando não ficam boas. O cara de negócio sabe o que precisa, mas não tem noção da **potencialidade técnica/UX**; é esse encontro que se busca.

E uma disciplina de método, válida principalmente para o pessoal de tecnologia:

> [!quote] Mergulhe no problema antes da solução
> "Quando começa a conversar sobre uma especificação, a gente já busca solução na cabeça — penso numa tabela, numa classe; a Júlia pensa num botãozinho — e **paro de ouvir**. Primeiro mergulhe no problema, entenda tudo; **só depois** pense nas caixinhas (e aí use a IA)."

## 🎓 Experiência, repertório e o "cinto de utilidades"

- Uma entrevista (Vale do Silício) defendeu que **experiência nunca foi tão valiosa**: quanto maior o **repertório**, mais se consegue **extrair da IA**. O júnior tem repertório limitado; o sênior extrai muito mais. "Para chegar nesse nível, tem que comer grama."
- O treinamento que o Andre gerou para o time prova o ponto: não é "só o prompt" — é a **experiência dele** que gera um documento bom (mapa conceitual, terminologias do Pluggy — item/conector, diagramas, exercícios no estilo **Feynman**, com a personagem fictícia **Marina**, a tesoureira).
- Recado da aula para devs: **subir de nível** — estudar **disponibilidade, segurança, arquitetura** (ampliar o "cinto de utilidades do Batman"), porque a IA assume o trabalho bruto de código.
- Os jovens têm a **vantagem da velocidade**: podem ganhar em pouco tempo, com a IA, a experiência que levou ~20 anos.

## 🏗️ Modelagem técnica — read model, eventos e Open Finance

### Modelo de leitura × integridade
O Claude trabalha com **read model**: enquanto não houver problema de performance, **lê direto na origem** (bank transaction / cash transaction). A distinção:

- Modelar para **integridade** (constraints, restrições) é bom para o **operacional/transacional** — garante que nada entra errado.
- Modelar para **leitura** nem sempre é mais rápido em cima do transacional; então se separa o sistema em **slices verticais**.

### Eventos e réplica (CQRS-ish)
Se uma vertical precisar de dado de outra **com performance**, entra a ideia de **réplica** (tabela enxuta só com o necessário), atualizada por **eventos**: a origem registra a transação → **gera evento** → o consumidor lê o evento na **frequência que precisar** e atualiza. Acoplamento assíncrono, não online. Conecta com [[modelagem-orientada-a-eventos]].

> [!example] Precedente real — o portal (ano passado)
> Para ler pagamentos, criaram uma **tabela fato** no mesmo banco. Sem volume em homologação, não sabiam se haveria gargalo. Em produção, ler direto de **Move Bancária / Move Compromisso** levava "vários minutos"; com a tabela fato, **caiu para segundos**. A atualização por evento: a Lei (analista) e a equipe **varreram as triggers/procedures** que fazem insert/update/delete nas tabelas Move e fizeram **gerar um evento** (uma tabelinha: data, hora, contador, registro alterado). Um processo lê essa tabela **de 10 em 10 minutos** e atualiza só o que mudou. Começar simples e subir a complexidade conforme a necessidade — "deixa o problema aparecer e resolve".

### Dúvidas sobre o Open Finance / Pluggy
O Mota compartilhou o "processo mental" da especificação (baseada na documentação do Pluggy + calibragem de negócio), com várias **incertezas a validar na prática**:

- **Transações bancárias** = fluxo **aditivo/incremental** (chegam registros novos). Dúvida: há distinção entre **movimentos fechados** (fechamento do dia anterior) e **intraday**? Para conciliar, usa-se **D-1**. Pode ser que a atualização "apague e reimporte tudo" — não está claro.
- **Investimentos** = **snapshot** (posição do dia): hoje uma posição, amanhã outra; guardam-se snapshots para análises (variação dia a dia). As **transações** vinculadas à posição podem ser aditivas — mas não se sabe se o Pluggy reenvia todas no JSON.
- **Granularidade de atualização** = **por item/conexão**, não por conta: uma conexão (ex.: Bradesco com 10 contas) tem **uma única data/hora de "última atualização"** para todas as contas. O Pluggy chama a conexão de **item**.
- **Performance**: o Andre avalia que **não deve ser gargalo** (carga histórica uma vez, depois atualizações diárias), mesmo com centenas/milhares de contas; ajusta-se conforme a necessidade.
- **Verbosidade do JSON**: campos adicionais vão num **JSON** (a tabela é quebrada em colunas declaradas + JSON de extras), o que é **verboso** (repete o nome de cada coluna) e trafega mais — possível ponto de ajuste técnico futuro.
- **AG Grid**: o Mota se inspirou no controle que aguenta **~100 mil registros** (consolidação de registros na transação bancária) — deve sobrar para o uso diário.
- **Agendamento**: se haverá **schedule forçado** ou o usuário submeterá a atualização — fica para depois.

## 🎯 Síntese / próximos passos

- **Implementar o mínimo (YAGNI)**: empresa + hierarquia + conta corrente (sem vínculo inicial) + banco + usuário mínimo; moeda só BRL. Objetivo: **fazer o Open Finance funcionar** com as estruturas mínimas.
- **Versão de experimentação** mirando a **semana do dia 3** para o Mota colocar dados reais e validar o comportamento da API do Pluggy.
- **Recursos liberados**: Clayton em intensidade máxima, assinaturas do Claude renovadas, sem economizar créditos.
- **Férias**: Andre provavelmente sai em **27/julho** (entrega até ~17), dividindo 7 + 7 com a outra metade em agosto.
- **Pessoas**: aproximar **Júlia e Rodrigo** do negócio (produto + UX); manter os **15 min/dia** mostrando código e documentação ao time.
- **Arquitetura**: começar simples (read model na origem), deixar a porta aberta para **eventos/réplica** quando a performance exigir.

## 🔗 Notas relacionadas
- [[2026-06-10-resumo-aprofundado]] — conversa anterior com o Mota (visão de produto e telas)
- [[open-finance]] · [[referencia-pluggy]] · [[referencia-universe]]
- [[cockpit]] · [[modelagem-orientada-a-eventos]] · [[ddd-domain-driven-design]]
- [[referencia-aulas-elemar-masterclasses]]
