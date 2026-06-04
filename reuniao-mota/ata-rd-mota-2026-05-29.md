## Ata detalhada

**Reunião:** RD Mota  
**Data:** 29 de maio de 2026  
**Objetivo:** detalhar a proposta de entregas do **Q2** para Open Finance/Tesouraria, cobrindo conectividade bancária, master data, transações, investimentos, dashboards executivos e possíveis frentes de evolução.

### 1. Contexto geral
- Foi apresentada uma visão reversa do fluxo de Open Finance para validar a estrutura da solução.
- A proposta parte de uma tela inicial de Open Finance “vazia”, em que o usuário conecta banco, autentica, concede consentimento e, a partir disso, o sistema passa a trazer dados de **contas correntes**, **transações** e **investimentos**.
- Foi destacado que ainda existem incógnitas sobre a totalidade dos dados disponíveis via integração, mas esses três blocos foram apontados como o núcleo prioritário para o Q2.

### 2. Proposta funcional para o Q2
#### 2.1 Open Finance e conectividade
- A jornada prevista começa com a conexão bancária, autenticação e consentimento do usuário.
- Após a conexão, o sistema atualiza cadastro e contas, e passa a disponibilizar os dados financeiros trazidos pela integração.

#### 2.2 Master data de contas
- Foi proposta a criação de um **master data de conta bancária** desacoplado inicialmente da empresa, para evitar dependência de deduções automáticas imperfeitas.
- A lógica sugerida é permitir cadastro de conta mesmo sem todas as vinculações completas, com posterior triagem e enriquecimento dos dados pelo usuário.
- O sistema deve trazer o que conseguir automaticamente da integração e sinalizar pendências, como ausência de vínculo com empresa.
- Foi descrito um status de **“descoberta”** para registros de contas identificados automaticamente via Open Finance, permitindo que novas contas sejam encaminhadas para tratamento sem bloquear o processo operacional.
- O usuário faria a manutenção dessas contas em uma triagem, atribuindo empresa e outros metadados faltantes.

#### 2.3 Transações bancárias
- Foi proposta uma tela analítica de **transações bancárias**, com painel de parâmetros e seleção dimensional já estruturada.
- A navegação deve evitar excesso de combos independentes, adotando hierarquia entre empresa, banco e conta para facilitar filtragem e coerência dos dados.
- Como primeira entrega, o sistema deve permitir ao usuário visualizar extratos e detalhes bancários trazidos da integração, com recursos de filtragem, consulta de saldo e detalhe por conta.
- A partir desses movimentos, foi sugerida a geração de visões mais consolidadas de tesouraria, incluindo saldos por conta e informações executivas atualizadas a partir da tabela de movimentos.

#### 2.4 Investimentos
- Foi apresentada uma tela para listar os **investimentos** trazidos da integração, com base nos campos disponíveis na documentação da fornecedora.
- A proposta considera que cada operação de investimento pode vir acompanhada de parcelas, juros e demais detalhamentos, o que abre possibilidade futura de separar telas de operações e parcelas.
- No curto prazo, a tela deve permitir visão detalhada dos registros recebidos e consulta das informações operacionais disponíveis.

#### 2.5 Visão gerencial de investimentos
- Além da visão analítica, foi proposta uma visão **gerencial/consolidada** de investimentos.
- Essa consolidação incluiria agrupamentos por banco, modalidade, empresa e vencimento, com faixas como até 7 dias, 8 a 30 dias e 30 a 90 dias.
- Também foram mencionados indicadores como total investido, valor líquido, disponível para resgate, valores a vencer nos próximos 30 dias e taxa média de retorno consolidada.
- Foi registrado que ainda há incerteza sobre o que efetivamente virá da integração e se será necessário complementar parte desses cálculos.
- Foi sugerido ainda um indicador de concentração e algum status de atualização/sincronização dos dados.

### 3. Alertas, exceções e evolução futura
- Como evolução, foi sugerido que o usuário possa criar **regras de monitoramento** para vencimentos, concentração e limites, gerando alertas e exceções sobre o portfólio.
- A ideia é que, enquanto o módulo mais completo de aplicações não estiver entregue, já exista algum mecanismo simples e de alto valor agregado para monitorar o que vem dos bancos.
- Esses alertas poderiam enriquecer o painel gerencial central.

### 4. Estrutura de cadastros e referências
#### 4.1 Entidades previstas
- Foi mencionado que o master data precisará contemplar **companhias**, hierarquias empresariais, bancos, contas correntes, moeda, calendário e instrumentos bancários.
- Alguns cadastros de menor complexidade foram pensados como referências simplificadas; outros, de maior complexidade, terão cadastro individualizado.

#### 4.2 Moeda
- O cadastro de moeda foi pensado com base na ISO, com baixa necessidade de manutenção manual pelo usuário.
- Cotação foi mencionada como tema separado, ainda não detalhado.

#### 4.3 Países e governança
- Também foi mencionada a intenção de manter lista de países baseada em ISO.
- Foi citado que, pensando em governança futura, os cadastros devem prever informação de responsável pela curadoria e manutenção, ainda que isso não tenha impacto imediato na primeira entrega.

### 5. Modelo de calendário
- Houve discussão específica sobre a modelagem de **calendários**.
- A proposta é ter calendários-padrão por país e permitir calendários específicos subordinados, herdando feriados nacionais e acrescentando apenas exceções locais.
- O exemplo dado foi o uso do calendário do Brasil como base e da B3 como calendário específico que herda os feriados nacionais e adiciona suas exceções próprias.
- Também foi discutida a possibilidade de calendários locais, como São Paulo, herdando do Brasil e adicionando apenas feriados próprios.
- O benefício central apontado foi evitar duplicação de cadastros e replicação de feriados de vários anos.
- Foi defendido também que calendários públicos relevantes, como o da B3 e de bolsas internacionais, sejam oferecidos automaticamente pelo sistema, sem exigir cadastro manual do usuário.

### 6. Atribuição de calendário
- Foi discutido que calendários têm vida própria e podem ser associados a diferentes dimensões, como empresa, conta corrente, índice e movimentos.
- Foi observado que, quando não houver conta corrente vinculada, o sistema pode usar o calendário da empresa como padrão para sugerir tratamento de datas úteis ou não úteis.
- Para contas correntes, o calendário atribuído à própria conta deve prevalecer.
- Também foi reforçada a necessidade de tratar corretamente movimentos em dias não úteis, com regras configuráveis de postergação ou efetivação conforme instrumento.

### 7. Dashboard executivo / Control Tower / Treasury Home
- Foi discutido que o **executive dashboard** faz parte do escopo de Q2, especialmente dentro do conceito de **Control Tower**.
- A visão inicial deve focar apenas nas funcionalidades realmente entregues no período, sem incluir componentes ainda fora de escopo.
- Foram mencionados elementos possíveis do painel: prioridades/insights, ações executáveis, chat resumido, KPIs para “ficar de olho”, links de exploração e blocos específicos de visão mensal ou distribuição de saldos.
- No entanto, foi explicitado que parte desses componentes ainda não entra agora; para o Q2, a recomendação foi exibir apenas o que estiver efetivamente disponível, como KPIs executivos, indicadores de investimentos e links para as telas já entregues.
- Também foi mencionado que o chat pode entrar de forma reduzida se houver viabilidade, mas isso não foi tratado como promessa.

### 8. Escopo prático do Q2
- Na consolidação verbal, foi indicado que o Q2 deve abranger:
  - conectividade/open finance;
  - telas de contas e transações;
  - master data necessário para suportar companhias, bancos, contas e referências;
  - tela e KPIs de investimentos;
  - dashboard executivo com recorte aderente ao que estiver realmente pronto.

### 9. Abordagem de execução
- Foi reconhecido que há bastante coisa no escopo e que o trabalho deve avançar **parte por parte**, começando pelas contas.
- Também foi dito que a primeira versão será iterativa e sujeita a ajustes conforme a implementação avançar.

### 10. Envolvimento de Rodrigo e Julia
- No fim da conversa, foi levantada a possibilidade de **Rodrigo e Julia** começarem a apoiar o processo.
- Foi sugerido que eles já poderiam começar olhando o código, estudando a estrutura pronta e entrando gradualmente no contexto de master data, telas ou chat.
- Também foi comentado que a implementação exigirá APIs por trás das telas, especialmente para entradas/saídas, investimentos e transações.

### 11. Encaminhamentos observáveis na conversa
- Anotar que o **executive dashboard / painel dentro do Control Tower** está previsto para o Q2.
- Começar a implementação pelas frentes de **contas** e entender sua estrutura antes de avançar para o restante.
- Avaliar o envolvimento inicial de Rodrigo e Julia na base de código e no entendimento das frentes que serão implementadas.

Se quiser, posso também:
1. **transformar isso em ata executiva de 1 página**, ou  
2. **organizar em “decisões / dúvidas / próximos passos”**.