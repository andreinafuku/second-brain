### Anotação detalhada da aula

#### 1. Documentação de projeto e “segundo cérebro”

Na parte final, começou uma sessão de perguntas com foco em práticas modernas de desenvolvimento e uso de IA.

Um dos participantes perguntou se manter documentação do projeto dentro do próprio repositório, como memória do histórico e contexto, se relaciona com a ideia de **“segundo cérebro”**.

##### Resposta trazida:
- o conceito de **Second Brain** foi associado ao trabalho de **Tiago Forte**
- a ideia central é criar mecanismos de anotação e organização para apoiar memória e recuperação de conhecimento
- foi colocado que os conceitos **não são iguais**, mas **têm semelhanças**
- houve recomendação explícita do livro **Building a Second Brain** como referência útil.

#### 2. Microsserviços, coesão e contexto para IA

Outro tópico foi o uso de IA em ambientes com microsserviços.

##### Questão levantada:
Como dar contexto suficiente para a IA quando o sistema está dividido em múltiplos serviços?

##### Pontos discutidos:
- pode fazer sentido criar um **agente orquestrador** que:
  - baixe os repositórios
  - consolide contexto
  - responda perguntas com visão ponta a ponta
- porém, houve um alerta arquitetural importante:
  - se o desenvolvimento de um microsserviço depende de muito mais do que o próprio contrato exposto, isso pode indicar **baixa coesão**
  - ou até um **monólito distribuído**, em vez de microsserviços bem definidos.

Também ficou claro que:
- esse tema exige discussão mais profunda
- coesão é fácil de explicar, mas difícil de sustentar na prática ao longo do tempo.

#### 3. ADRs versus Guidelines

A aula diferenciou dois tipos de documentação técnica:

##### ADR (Architecture Decision Record)
Usado quando há:
- decisão arquitetural relevante
- mudança estrutural importante no projeto
- introdução de componentes novos, como por exemplo cache

##### Guidelines
Usadas quando:
- o time precisa alinhar práticas recorrentes do dia a dia
- há decisões mais próximas de design e implementação
- não se trata de uma grande decisão arquitetural formal.

A distinção apresentada foi:
- **ADR** = decisão arquitetural importante
- **Guideline** = direcionamento operacional/de design para execução cotidiana.

#### 4. Hierarquia documental: PRD, ADR e Guidelines

A discussão avançou para a relação entre documentos de diferentes níveis.

##### Ideia central:
A documentação forma uma hierarquia, em que documentos mais altos orientam os mais específicos:

- **PRD**: contexto e verdade em nível de negócio
- **ADR**: decisões arquiteturais derivadas
- **Guidelines**: práticas operacionais derivadas dessas decisões

Também foi destacado que:
- o negócio muda
- então não existe “verdade absoluta” imutável
- o PRD precisa ser **atualizado, versionado, entendido e revisado** conforme o contexto evolui.

Outro ponto importante:
- documentos de nível mais alto tendem a mudar **menos frequentemente**
- documentos mais próximos da implementação tendem a mudar **mais**.

#### 5. Trade-off central: velocidade inicial vs manutenção futura

Esse foi um dos pontos mais fortes do encerramento da aula.

##### Princípio apresentado:
Existe um trade-off entre:
- **acelerar a primeira versão**
- **acelerar a manutenção e evolução futuras**

##### Formulação do raciocínio:
- se você prioriza entregar muito rápido no início, tende a aumentar o custo de manutenção depois
- se você investe mais cedo em estrutura, documentação e organização, aumenta o setup inicial, mas reduz o custo futuro.

##### Aplicação prática com IA:
- gerar PRDs, ADRs, guidelines e estrutura documental dá trabalho no começo
- mas isso cria base para que a IA consiga desenvolver com muito mais velocidade depois
- a documentação não substitui o desenvolvimento: ela **amplifica a velocidade futura de implementação**.

A mensagem final foi:
- no início parece que se está “escrevendo documentação em vez de código”
- mas, com o tempo, a documentação estabiliza
- enquanto o código continuará crescendo indefinidamente
- por isso, uma boa documentação passa a funcionar como acelerador permanente do desenvolvimento.

#### 6. Encerramento e direcionamento da mentoria

No fechamento, foi explicado que:
- vários conceitos apresentados ainda são avançados
- isso é esperado
- os fundamentos serão trabalhados nas próximas mentorias para tornar as práticas mais compreensíveis

Também foi reforçado que:
- a proposta não é fazer um “cursinho”
- mas expor prática real, com exemplos reais
- mesmo que isso torne a absorção inicial mais exigente.
