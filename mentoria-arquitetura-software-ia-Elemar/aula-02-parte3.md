## Ideia central: o “problema dos Jetsons”

Elemar começou retomando a analogia entre **Jetsons** e **Flintstones**. A provocação é que, embora um desenho se passe no futuro e o outro no passado, a **dinâmica dos processos é praticamente a mesma**. A crítica é que a tecnologia muda, mas muitas vezes as pessoas continuam operando do mesmo jeito.

A aplicação disso para IA é direta: **usar IA para repetir exatamente os mesmos processos antigos tende a gerar apenas ganho marginal de eficiência**. O ganho real aparece quando o time revê o processo em si e decide o que ainda faz sentido continuar fazendo e o que precisa ser transformado.

O alerta de Elemar foi para evitar o “problema dos Jetsons”: **manter dinâmicas antigas apenas porque sempre funcionaram assim**, mesmo quando a nova tecnologia permitir uma abordagem melhor.

---

## Regra 10-80-10

Uma das ideias mais enfatizadas na aula foi a regra **10-80-10** para desenvolvimento com IA.

### 1) Primeiros 10%
É o momento de:
- ler o problema;
- entender a necessidade;
- revisar os documentos;
- analisar a especificação;
- pensar no que está faltando.

Esse começo é importante porque define a qualidade do que será produzido depois. A orientação foi clara: **não sair pedindo para a IA gerar código sem antes fazer esse trabalho inicial de entendimento**.

### 2) 80% centrais
Aqui entra a IA executando grande parte do desenvolvimento:
- geração de código;
- transformação de documentação;
- criação de estruturas;
- apoio na implementação.

Mas isso só funciona bem quando os 10% iniciais foram bem feitos.

### 3) 10% finais
É a etapa de:
- validação;
- navegação no resultado;
- conferência do comportamento;
- verificação se a saída realmente atende ao que era esperado.

A recomendação foi não usar a IA no fluxo “gera → não gostei → gera de novo” de forma aleatória. O ideal é **investir na especificação antes**, para elevar a chance de a primeira geração já vir muito mais próxima do correto.

---

## Estocástico vs determinístico

Outro ponto central da aula foi a diferença entre **IA estocástica** e **especificação determinística**.

### IA é estocástica
Segundo Elemar, a IA é estocástica por natureza:
- resultados variam entre execuções;
- a mesma solicitação pode gerar respostas diferentes;
- não existe garantia de repetição exata do mesmo código.

Ou seja, não se deve esperar previsibilidade absoluta da geração.

### Especificações devem ser determinísticas
Em contraste, o que o time produz como base precisa ser claro e preciso:
- o que deve ser feito;
- quais restrições existem;
- quais critérios precisam ser atendidos;
- quais padrões arquiteturais devem ser seguidos.

A ideia é que **a fonte de verdade seja determinística**, mesmo que a geração seja estocástica.

### Mudança de paradigma
A aula também trouxe uma mudança importante de mentalidade:
- antes, o foco era especificar detalhadamente **como fazer**;
- agora, ganha força a ideia de mecanismos para verificar **se o resultado está correto**.

Isso não elimina especificação, mas desloca parte da atenção para **verificabilidade**.

---

## ADRs como fonte da verdade

Gabriel mostrou que os **ADRs (Architecture Decision Records)** estão sendo usados como a base canônica do projeto.

### Função dos ADRs
Os ADRs servem para registrar:
- o contexto;
- a decisão tomada;
- alternativas consideradas;
- consequências da escolha.

Na demonstração, isso aparece como a **fonte da verdade** para orientar ferramentas, agentes e regras do projeto.

### Papel na engenharia com IA
A lógica apresentada foi:
- primeiro, registrar decisões de arquitetura;
- depois, usar essas decisões como base para gerar outros artefatos;
- por fim, fazer a IA trabalhar a partir dessa base, em vez de improvisar do zero.

A aula reforçou que, sem esse tipo de base arquitetural, fica muito mais difícil usar IA com consistência em projetos reais.

---

## Diferença entre ADR e guideline

Foi feita uma distinção importante entre **ADR** e **guideline**.

### ADR
O ADR:
- registra a decisão arquitetural;
- documenta o porquê da decisão;
- opera em nível mais estratégico.

### Guideline
A guideline:
- traduz a decisão para o dia a dia;
- explica como implementar ou aplicar aquela decisão;
- serve como orientação prática para desenvolvimento e validação.

A forma como isso foi apresentada sugere uma hierarquia:
1. a decisão é registrada no ADR;
2. essa decisão é desdobrada em guideline;
3. a guideline pode alimentar regras, prompts, agentes e validações automáticas.

---

## Documentação para IA vs documentação para humanos

Elemar trouxe uma observação prática importante: em muitos contextos, **ninguém lê o resumo gerado**, e o que acontece de fato é a IA ler o resumo e produzir um novo resumo. Isso leva a um questionamento sobre **para quem a documentação está sendo feita**.

### Dois tipos de documentação
A aula apontou para a necessidade de distinguir:
- documentação feita para humanos;
- documentação feita para IA.

### Documentação voltada para IA
Quando a documentação é pensada para IA, ela pode exigir:
- formato mais estruturado;
- conteúdo mais parseável;
- menor custo de tokens;
- melhor legibilidade para máquinas do que para pessoas.

Foi citado o raciocínio de que, em alguns casos, um formato mais estruturado pode ser mais adequado do que formatos tradicionais voltados à leitura humana.

### Implicação prática
O time precisa começar a identificar:
- quais artefatos continuam sendo prioritariamente humanos;
- quais artefatos passam a existir principalmente para serem consumidos por LLMs.

---

## Arquitetura de agentes especializados

Outro bloco forte da aula foi a discussão sobre **agente único vs múltiplos agentes especializados**.

### Problema do agente único
Elemar comentou a experiência com uma agente chamada Márcia, que começou como um agente único. O problema percebido foi:
- excesso de responsabilidade concentrada;
- instruções incompatíveis dentro do mesmo agente;
- dificuldade de manter consistência em uma estrutura grande demais.

### Solução: fracionar
A evolução foi dividir esse agente em agentes menores:
- cada um com função mais delimitada;
- responsabilidades específicas;
- compartilhamento de uma memória global, mas com atuação separada.

### Analogia com gestão de pessoas
A comparação feita foi que agentes passam a ter algo próximo de:
- descritivos de função;
- competências específicas;
- papéis comparáveis aos de especialistas humanos.

A ideia é que, assim como pessoas são organizadas por competências e papéis, os agentes também podem ser organizados dessa forma.

---

## Pessoas, agentes, processos e tecnologia

Elemar propôs uma atualização na forma de pensar organizações.

### Modelo antigo
- pessoas;
- processos;
- tecnologia.

### Modelo novo
- pessoas;
- agentes;
- processos;
- tecnologia.

O ponto principal é que agentes passam a ser tratados como parte real da força de trabalho. Isso implica:
- definir quais agentes a organização precisa;
- delimitar responsabilidades;
- entender como esses agentes colaboram com pessoas e processos.

Na fala dele, isso também se conecta com especialistas de domínio e com a estrutura organizacional das empresas.

---

## Verificabilidade vs especificação

A aula trouxe a ideia de que, no desenvolvimento moderno com IA, pode ser mais importante ter **formas baratas e confiáveis de verificar o resultado** do que tentar descrever exaustivamente cada detalhe de implementação.

### O foco muda
Antes:
- especificar minuciosamente como fazer.

Agora:
- construir meios de verificar se aquilo que foi entregue atende ao que precisa ser atendido.

### Exemplos práticos citados
Foram mencionados, nesse contexto:
- critérios de aceite;
- fitness functions;
- validações arquiteturais;
- testes ligados a regras de negócio;
- métricas de performance.

O objetivo é permitir que a geração seja flexível, mas ainda assim governada por mecanismos claros de validação.

---

## Fluxo prático mostrado por Gabriel

Gabriel mostrou um fluxo estruturado de trabalho com IA dentro do projeto.

### 1) Memória do produto
Ele apresentou um agente com memória do produto, contendo:
- conhecimento do domínio;
- materiais;
- avaliações;
- registros de interações;
- contexto acumulado sobre o produto.

Essa memória serve como base para geração de artefatos.

### 2) Geração de PRD
A partir dessa base, é gerado um **PRD** voltado para o produto, funcionando como uma especificação mais ligada ao problema de negócio.

### 3) Design
Depois do PRD, entra a etapa de design, usada para refinar a proposta e explorar como a solução será apresentada ou estruturada.

### 4) Épicos
O PRD é quebrado em épicos. A ideia mostrada foi evitar histórias grandes demais e dividir o trabalho em blocos que gerem valor.

### 5) User stories
Os épicos são detalhados em histórias menores, com a orientação de que essas entregas sejam pequenas.

### 6) Implementação com IA
A IA implementa com base em:
- ADRs;
- guidelines;
- histórias refinadas;
- critérios claros.

### 7) Testes
Os testes são gerados a partir do que foi especificado e do que precisa ser validado no negócio.

A lógica mostrada foi de um pipeline em que a geração não começa pelo código, mas por uma sequência de artefatos e refinamentos.

---

## Exemplo prático apresentado

Um dos exemplos discutidos foi o de um motor voltado a score/sinais operacionais para condução de dailies com base em dados, e não apenas nas perguntas tradicionais.

A ideia apresentada foi que a daily deveria ser orientada por sinais como:
- itens parados por muitos dias;
- acúmulo excessivo de trabalho em uma pessoa;
- desvios de tempo em relação ao histórico do time.

Nesse contexto, um detalhe pequeno na especificação — como dizer que o motor deveria ser extensível — alterava significativamente o resultado da implementação. A lição aí foi que **uma pequena mudança bem colocada na especificação pode mudar toda a qualidade do código gerado**.

---

## QA e testes baseados em negócio

Gabriel comentou também a visão dele sobre QA e testes.

### Mudança percebida
A abordagem que ele relatou é:
- implementar a funcionalidade;
- depois gerar plano de testes mais alinhado ao negócio.

### Qualidade não é só cobertura técnica
A crítica foi contra uma visão limitada de cobertura, em que basta “passar por todas as linhas”. O ponto defendido foi que teste bom precisa verificar:
- regras;
- cenários reais;
- condições relevantes do domínio.

### Princípio importante
A qualidade dos testes que a IA escreve depende da qualidade da engenharia que o time já sabe fazer. Ou seja:
- se o time não sabe definir bons testes,
- não adianta esperar que a IA compense isso sozinha.

---

## Conselhos para iniciantes

A aula trouxe vários conselhos práticos para quem está começando.

### 1) Não cair no copy-paste
Foi reforçado que não basta copiar prompt, estrutura ou repositório sem entender o raciocínio por trás.

### 2) Não corrigir código manualmente como primeira reação
Quando a IA gera algo ruim, a recomendação foi:
- não sair corrigindo direto no código;
- voltar para a especificação;
- entender o que faltou orientar;
- gerar novamente com base ajustada.

### 3) Construir maturidade
Foi deixado claro que esse tipo de trabalho exige prática. Parte da habilidade vem de:
- ler melhor a especificação;
- perceber lacunas;
- aprender a orientar melhor a IA.

### 4) Foco nos fundamentos
A recomendação final foi manter a atenção nas ideias mais importantes:
- 10-80-10;
- estocástico vs determinístico;
- verificabilidade;
- papel de ADRs;
- importância da arquitetura antes da geração.

---

## Combinados finais do grupo

No encerramento, o grupo alinhou alguns próximos passos.

### Rever a aula
A percepção geral foi que a aula foi densa e que **vale rever o conteúdo** para absorver melhor os conceitos.

### Prioridade nos fundamentos
A orientação foi não tentar absorver todos os detalhes técnicos de uma vez, mas sair com os princípios principais.

### Continuidade
Foi combinado continuar acompanhando:
- as próximas aulas;
- os materiais;
- as discussões no Circle;
- os exemplos práticos que Gabriel ainda deve mostrar em outro encontro.
