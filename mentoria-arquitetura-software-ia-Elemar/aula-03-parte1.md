## Anotação completa

- **Tema central:** arquitetura como exercício de **modelagem** do mundo real por meio de simplificações úteis. Modelo não é a realidade; é uma abstração para explicar e operar sobre ela. O **DDD** entra como uma **filosofia** para descobrir quais elementos do domínio importam e como representá-los.

### 1. Modelo, abstração e arquitetura
- Todo modelo é uma **simplificação** e exige **abstração**. Não existe forma única de representar o mundo; diferentes times podem explicar o mesmo negócio de formas diferentes.
- O grande trabalho de arquitetura é a **descoberta do modelo**: identificar quais elementos importam, como eles se relacionam e quais partes governam o restante do sistema.
- O exemplo da Ambev foi usado para mostrar que nem sempre é claro onde uma empresa “ganha o jogo” — logística, distribuição, produto etc. O arquiteto não necessariamente encontra a verdade absoluta, mas precisa ajudar a construir uma **resposta de consenso**.

### 2. DDD estratégico, core e bounded contexts
- Quando um conjunto de componentes representa o **core** do negócio, o restante deveria estar subordinado a ele; as relações entre contextos importam porque indicam quem está no **upstream** e quem sofre impacto de mudança.
- O professor defende, como filosofia, que **microservice = bounded context**, embora reconheça que isso não é consenso universal.
- Dessa visão deriva a ideia de que:
  - um **time pode manter vários contextos**;
  - mas **um contexto deveria ser mantido por um único time**;
  - por inferência, um time pode manter vários microsserviços, mas um microsserviço não deveria ser mantido por vários times.
- Isso foi conectado à **Lei de Conway**: a estrutura de comunicação da organização tende a determinar a arquitetura do sistema. Se dois times mexem no mesmo componente, aumentam ruído, fragmentação de visão e acoplamento.

### 3. Linguagem ubíqua e descoberta de contexto
- O DDD ajuda a descobrir contextos por meio da conversa com **especialistas de domínio**, buscando a **linguagem ubíqua/onipresente**.
- Um sinal forte de fronteira de contexto é:
  - especialistas diferentes usando **termos diferentes** para a “mesma coisa”; ou
  - usando o **mesmo termo** para coisas diferentes.
- Exemplo do hospital:
  - na enfermaria, a pessoa é **paciente**;
  - na tesouraria/financeiro, é **cliente**;
  - no plano de saúde/seguro, pode ser **vida** ou **segurado**.
- A conclusão é que esses nomes não são mera convenção superficial: cada um revela um **contexto diferente**, com significados, atributos e regras próprios.

### 4. Entidades, value objects e contexto
- **Entidade** foi descrita como uma coisa com **identidade**. Parte do esforço de modelagem é descobrir quais entidades compõem o domínio.
- Essas entidades só fazem sentido quando observadas em determinado **contexto**. Quem ajuda a delimitar esse contexto é o especialista de domínio e sua linguagem.
- O DDD também distingue coisas sem identidade própria, tratadas como **value objects / objetos de valor**.

### 5. DDD estratégico vs. tático
- Houve discussão sobre se descobrir o core e o domínio seria “mais importante” do que a modelagem tática. A resposta foi **depende**.
- Para decisões mais altas de arquitetura, o **DDD estratégico** costuma dar mais insight do que detalhes táticos como entidades, value objects e repositórios.
- Ainda assim, o nível tático não pode ser ignorado, porque decisões como **persistência** podem depender fortemente da forma como as entidades são modeladas.

### 6. Modelagem e persistência
- Exemplo: um marketplace como a Amazon, com tipos de produto muito diferentes, pode gerar dificuldade de modelagem em banco relacional, porque os atributos variam muito entre categorias. Isso pode levar a escolher um modelo de persistência diferente do relacional.
- A mensagem foi que não dá para pensar o domínio “sem pensar na persistência” por muito tempo: persistência também é uma **visão do modelo**.
- O banco de dados não é neutro; ele expressa parte da forma como o sistema vê o domínio.

### 7. Integração entre contextos e separação de dados
- O professor destacou que o maior calcanhar de Aquiles dos sistemas organizacionais são as **integrações**. Por isso ele valoriza o **diagrama de contexto do C4**.
- Problemas como sincronia de dados entre microsserviços podem indicar **modelagem errada dos contextos**.
- No exemplo do hospital, ele rejeita uma tabela única `Pessoa` servindo ambulatório, financeiro e plano de saúde, porque isso criaria:
  - acoplamento excessivo entre contextos;
  - mistura de atributos com significados diferentes;
  - risco de vazamento de informação indevida entre áreas.
- Exemplos de diferenças entre contextos:
  - no ambulatório importam certos dados clínicos;
  - no financeiro importa como a pessoa se identifica para faturamento;
  - no plano de saúde, compartilhar certos dados clínicos pode ser problemático.
- Mesmo quando paciente e cliente frequentemente coincidem, há exceções relevantes, como **recém-nascidos**, e essas exceções podem ou não ser importantes para o modelo — isso deve ser decidido no diálogo com especialistas.

### 8. DDD não é para “desenvolver um sistema isolado”
- Um ponto forte foi que o DDD é usado para entender o **domínio da organização como um todo**, e não apenas “o domínio deste programa”.
- Subdomínio e bounded context são propriedades do **negócio**, não do programa em si. Os sistemas são recortes que suportam partes desse domínio maior.

### 9. DDD pode aumentar complexidade?
- Pergunta levantada: DDD aumenta complexidade? Resposta: **DDD vira problema quando é usado como mecanismo de prescrição rígida**, em vez de filosofia para atacar a complexidade no coração do software.
- A crítica principal foi ao uso dogmático de padrões táticos e “máximas” repetidas sem entender os princípios.

### 10. Crítica ao dogma de repositórios e camadas
- Foi criticada a ideia de que “**acesso ao banco só pode ser feito via repositório**”. O professor disse que isso costuma ser tratado como verdade absoluta sem base clara e pode gerar implementações absurdamente ineficientes.
- Exemplo dado: reajustar preços de todos os produtos de um fornecedor.
  - abordagem dogmática: carregar milhares de produtos, iterar em memória, atualizar um a um, usar unit of work etc.;
  - alternativa mais simples: executar uma operação direta no banco para atualizar em lote.
- A crítica não foi ao DDD em si, mas ao uso cego de padrões como repositório, unit of work, notifications e mediator quando eles só aumentam custo e latência.

### 11. Separação entre domínio e infraestrutura
- Outro alvo da crítica foi a regra automática de separar interfaces no domínio e implementações concretas na infraestrutura “para poder trocar o banco”.
- A posição foi:
  - **só faz sentido** se houver cenário real em que trocar o banco seja plausível ou necessário;
  - em muitos sistemas, isso é improvável e a abstração só adiciona custo.
- Exemplo em que faria sentido: produto/ERP que precisa suportar múltiplas bases para clientes diferentes.
- Também foi argumentado que abstrair vários bancos pode forçar o sistema a usar apenas o **mínimo denominador comum**, perdendo capacidades específicas de cada tecnologia.

### 12. Postgres e modelo menos anêmico
- Um participante relatou ter deixado parte da lógica no **Postgres**, por ser melhor para certos cálculos de horários/cobertura de médicos. O professor concordou e disse que o banco também expressa o modelo.
- A visão apresentada foi que ignorar o banco empobrece a visão de arquitetura; a persistência complementa a compreensão do modelo.

### 13. O que é modelo anêmico
- A discussão sobre **modelo anêmico** foi aprofundada. Um exemplo simples de classe `Employee` com `Id`, `Name`, `Cpf`, `Salary` e apenas `get/set` foi classificado como anêmico.
- O ponto central não é apenas “tirar setter” ou adicionar métodos genéricos como `AtualizarNome`. Isso não resolve a anemia por si só.
- A forma correta de enriquecer o modelo seria perguntar:
  - **quais são os motivos de mudança** de cada propriedade?
  - em quais **cenários de negócio** aquele estado muda?
- Exemplo:
  - salário não muda por um setter arbitrário;
  - ele muda por algo como **movimentação salarial**;
  - a movimentação pode ser **compulsória**, podendo incluir acordo coletivo ou dissídio.
- Nesse caso, o modelo passa a refletir operações do domínio com significado real, e propriedades podem ter alteração privada, mediada por comportamentos semânticos.

### 14. Eventos de domínio e event sourcing
- Nessa mesma linha, eventos de domínio foram apresentados como o **reconhecimento da ocorrência de uma operação** que justificou mudança de estado.
- Eles são especialmente úteis para informar **contextos satélites** interessados naquela mudança.
- Isso conecta com **event sourcing**, onde o histórico dos eventos explica tanto os estados quanto suas causas.
- Porém, novamente, a adoção depende de haver valor real nisso; lançar eventos sem consumidores pode ser só complexidade gratuita.

### 15. Testes e complexidade desnecessária
- Foi criticado o uso de testes unitários sobre estruturas triviais com apenas `get/set`, porque isso não testa lógica de negócio relevante.
- A defesa foi que testes valem mais quando o modelo expressa motivos de mudança e regras reais, porque aí há comportamento significativo para validar.
- Também houve crítica a testes de fluxo sem valor, em que se “moca” repositório apenas para verificar se um método chamou outro.

### 16. O que é decisão arquitetural
- Nem toda decisão de modelagem tática é “arquitetura”. Decidir se uma entidade específica está ou não anêmica não é, em geral, uma decisão arquitetural.
- Já decidir que, em certo contexto, o time deve **buscar modelos não anêmicos**, ou que **não haverá separação de infraestrutura**, ou que **será preciso suportar troca de banco**, isso sim é decisão arquitetural.
- Essas decisões devem ser justificadas por:
  - objetivos de negócio;
  - restrições;
  - atributos de qualidade;
  - custo e risco.
- Foi sugerido registrar esse tipo de escolha em uma **ADR**.

### 17. Regras vs. princípios
- Um dos blocos mais fortes da aula foi a distinção entre **regras** e **princípios**.
- A ideia:
  - se todos entendessem bem os princípios, muitas regras poderiam ser dispensadas;
  - como nem todo time tem essa maturidade, regras ajudam a evitar erro;
  - o problema começa quando a regra é seguida cegamente, sem entender o princípio que a justifica.
- Exemplos religiosos/culturais foram usados para ilustrar como regras podem sobreviver separadas de seus motivos originais.
- Aplicando isso a software: práticas como criar interface para todo repositório ou isolar infraestrutura por padrão podem ter surgido de bons princípios, mas se aplicadas mecanicamente podem virar custo inútil.

### 18. Acoplamento, coesão e trade-offs
- Foi recomendado estudar **acoplamento** e **coesão** como base para entender patterns e decisões de design.
- Mensagem principal:
  - queremos **baixo acoplamento**;
  - mas não à custa de destruir a **coesão**.
- Criar camadas e abstrações demais pode até reduzir certo acoplamento, mas também fragmentar demais o código e piorar a coesão.
- O arquiteto precisa equilibrar esse trade-off de forma consciente.

### 19. Complexidade desnecessária é custo
- Uma frase-chave da aula foi que **complexidade desnecessária é custo**.
- Mais elementos, mais camadas, mais abstrações e mais padrões do que o necessário aumentam o custo de mudança, leitura e manutenção.
- A função do arquiteto foi descrita como **evitar complexidade**, porque o valor econômico do software também vem da complexidade que se consegue não introduzir.

### 20. Hábito, dissonância cognitiva e crescimento
- No encerramento, a aula saiu um pouco do técnico e entrou em comportamento:
  - **dissonância cognitiva**: quando duas crenças que parecem verdadeiras entram em conflito;
  - crescimento ocorre ao tentar compatibilizar essas tensões ou revisar crenças.
- Também foi dito que muitas decisões humanas não são guiadas pelo que é melhor, mas pelo que é **habitual**. Isso vale para a vida e para o código.
- A mentoria foi apresentada como espaço para:
  - gerar dúvida produtiva;
  - quebrar certezas automáticas;
  - incentivar convivência com perspectivas diferentes.

## Ideias-chave da aula
- Modelo é simplificação útil, não espelho fiel da realidade.
- Arquitetura é, em grande parte, descoberta e explicitação de modelo.
- DDD é mais valioso como **filosofia de modelagem** do que como conjunto rígido de prescrições.
- Bounded contexts devem emergir da linguagem e dos especialistas de domínio.
- Time idealmente não compartilha a manutenção do mesmo contexto/microsserviço com outro time.
- Persistência também é visão de modelo e influencia arquitetura.
- Modelo anêmico não se resolve só escondendo setters; resolve-se representando **motivos reais de mudança**.
- Seguir regra sem entender princípio leva a complexidade inútil.
- Baixo acoplamento precisa ser equilibrado com alta coesão.
- Complexidade desnecessária é custo.

## Frases-força capturadas da aula
- Modelo é sempre simplificação do mundo.
- O grande exercício de arquitetura é a descoberta do modelo.
- DDD não deve adicionar complexidade ao coração do software; deve ajudar a atacá-la.
- O banco de dados também é uma visão do modelo.
- Regras sem princípios podem virar custo.
- Complexidade desnecessária é custo.

## Ações / próximos passos percebidos
- Revisar decisões arquiteturais do time perguntando sempre:
  - qual problema real isso resolve?
  - qual princípio justifica essa regra?
  - essa abstração reduz risco real ou só adiciona custo?
- Estudar/revisitar:
  - bounded contexts;
  - linguagem ubíqua;
  - coesão e acoplamento;
  - ADRs;
  - relação entre modelagem e persistência.
- Reavaliar uso automático de:
  - repositórios para tudo;
  - separação domínio/infraestrutura por padrão;
  - eventos e mediators sem necessidade clara.

Se quiser, eu também posso converter isso em um formato mais útil para estudo, como:

- **resumo executivo de 1 página**
- **anotações estruturadas por tópicos**
- **flashcards/perguntas e respostas**
- **mapa mental em markdown**