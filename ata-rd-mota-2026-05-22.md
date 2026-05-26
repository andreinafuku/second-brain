## Ata detalhada

**Reunião:** Reunião Mota e Carlos

**Data:** 22/05/2026

**Participantes identificados:** André Minoru Inafuku

**Observação:** reunião informal/presencial, com parte dos participantes não identificados explicitamente no registro.

---

### 1\. Abertura e contexto inicial

-   Houve uma breve conversa informal sobre recuperação física e rotina da semana antes de entrarem nos temas técnicos.

---

### 2\. Andamento da V2

-   Foi informado que a parte em andamento da V2 está sendo finalizada e que a expectativa é arredondar a parte de autenticação até a próxima sexta-feira, com base no que já está especificado hoje.
    
-   A implementação está sendo feita de forma mais resumida, encaixando a ferramenta escolhida para autenticação ao que já foi discutido.
    

#### 2.1. Definição da solução de autenticação

-   A solução comentada para autenticação foi o **Keycloak**, apresentada como uma alternativa externa e consolidada para gerenciamento de autenticação.
    
-   Foi explicado que:
    
    -   é uma solução open source;
        
    -   pode ser executada localmente ou em produção a partir de imagem/instalação;
        
    -   evita desenvolver manualmente fluxos de autenticação, fatores, geração e gestão correlata;
        
    -   já suporta cenários comuns de autenticação federada, como login por provedores externos;
        
    -   se encaixa no tipo de componente que não gera diferencial competitivo quando desenvolvido internamente, sendo melhor adotar uma solução pronta.
        
-   A percepção do grupo foi de que a escolha faz sentido e segue padrão amplamente usado.
    

---

### 3\. Funcionalidades já trabalhadas na implementação

-   Foi citado que um primeiro item, relacionado à inserção de transação de caixa, está simples de seguir porque depende majoritariamente das regras de negócio já definidas.
    
-   O segundo item discutido foi a listagem, incluindo preocupação explícita com requisito não funcional de desempenho.
    

#### 3.1. Testes automatizados e testes de performance

-   Foi explicado que requisitos de negócio estão sendo cobertos com testes automatizados, organizados separadamente no projeto, com cenários positivos e negativos, executados a cada modificação para evitar subida de erro ao repositório.
    
-   Já os testes de performance foram tratados de forma distinta:
    
    -   não foram colocados junto da suíte principal automatizada;
        
    -   isso ocorreu porque testes de carga podem demorar vários minutos e inviabilizar o fluxo cotidiano de desenvolvimento;
        
    -   foi criado um projeto/pasta separado de benchmark para rodar cenários de performance manualmente.
        

#### 3.2. Estrutura de benchmark e resultados

-   O benchmark descrito:
    
    -   sobe dependências via Docker;
        
    -   puxa imagem do Oracle;
        
    -   cria massa de testes;
        
    -   executa cenários derivados da especificação funcional/não funcional;
        
    -   mede resultados como percentil 95.
        
-   Foram mencionados quatro cenários oriundos da especificação, incluindo casos sem filtro, com intervalo de datas, filtrado por status e paginação.
    
-   No teste executado, dois cenários ultrapassaram o alvo de 200 ms no percentil 95, e isso foi registrado como resultado.
    
-   Também foi observado que seria importante repetir o teste com a máquina em melhores condições, pois o hardware/estado do ambiente local pode ter influenciado os números.
    

#### 3.3. Percepção do grupo sobre esse modelo

-   O grupo considerou valioso o fato de os requisitos de performance estarem registrados e rastreáveis no processo.
    
-   Foi destacado que testes de carga normalmente não são feitos pelo esforço exigido, mas que com esse apoio automatizado passa a ser mais viável executá-los.
    

---

### 4\. Status de integração/plugin com Cleiton

-   Foi informado que parte da implementação relacionada ao plugin/transaction não foi detalhada no mesmo nível de profundidade da outra frente.
    
-   O detalhamento ficou com Cleiton.
    
-   O que já está pronto:
    
    -   a conexão já é criada;
        
    -   já é possível entrar e usar o recurso;
        
    -   o ponto em verificação era a diferença entre uma documentação/conexão direta da plugin e outra abordagem usando o fluxo citado;
        
    -   depois disso, a intenção era testar com uma conta real.
        

---

### 5\. Discussão sobre performance no caso Apoc / fluxo de caixa

-   Foi relatado que houve conversa sobre um caso ligado a fluxos de caixa diários em período mensal na base da NEO.
    
-   Julia testou um cenário em que conta financeira, empresa e moeda funcionaram bem.
    
-   A degradação de performance apareceu quando começaram a ser adicionadas mais combinações/dimensões, incluindo casos com cerca de seis ou sete combinações na linha.
    
-   Foi citado também outro caso em análise envolvendo soma que não estava batendo.
    

#### 5.1. Encaminhamento sobre liberação

-   Foi discutido que já seria possível liberar a funcionalidade, desde que com ressalva clara sobre esse caso específico de fluxo de caixa com grande volume/variedade de combinações em dimensões de linha.
    
-   A visão defendida foi que não valeria esperar tudo ficar “100% redondo” para entregar.
    
-   Em contraponto/complemento, foi colocado que o mais importante é entender a expectativa do cliente e eventualmente restringir cenários na primeira versão, se necessário.
    
-   Também se comentou que os clientes que acabaram sobrando para esse contexto são mais exigentes/problemáticos, o que aumenta a pressão em torno de performance e parceria.
    
-   Foi reforçado que, apesar de nem todo o universo de combinações performar bem, o que antes não atendia já passou a atender nos cenários principais.
    
-   A decisão final ficaria dependente da conversa do Adolfo com o restante do pessoal ainda naquele dia.
    

---

### 6\. Revisão de código e mudanças em política de repositório

-   Foi informado que alguns revisores foram adicionados como obrigatórios no fluxo de revisão de código.
    
-   Houve menção a uma revisão recente solicitada por Alessandra nesse contexto.
    
-   Também foi relatado que a Microsoft mudou novamente as regras/políticas do repositório no produto atual, tornando incompatível uma política anterior.
    
-   A política antiga precisou ser retirada. Essa política exigia associação do código a uma feature/work item para permitir o retorno ao repositório.
    
-   Foi observado que:
    
    -   a política deixou de aparecer com as versões/ferramentas atuais;
        
    -   com ajuda externa foi possível remover a política antiga;
        
    -   por enquanto o repositório ficou sem essa checagem;
        
    -   para as pessoas mais habituadas ao processo isso não deve gerar grandes problemas imediatos.
        

---

### 7\. Banco Oracle 26 e ferramenta de acesso

-   Foi lembrado que as bases da NEO foram criadas em banco Oracle versão 26.
    
-   Surgiu a pergunta sobre também usar essa versão no desenvolvimento no futuro.
    
-   Foi relatado um problema prático:
    
    -   as máquinas usam SQL Navigator apoiado em cliente Oracle 11;
        
    -   essa versão não conversa com Oracle 26.
        
-   Para evitar mexer no cliente instalado do ambiente, foi sugerido usar o SQL Developer da Oracle, por ser portável e não exigir instalação tradicional.
    
-   Foi comentado ainda que o SQL Navigator usado é bastante antigo, enquanto a ferramenta da Oracle hoje atende melhor inclusive para administração.
    
-   Encaminhamento informal: quem precisar acessar a base da NEO deve baixar e usar essa alternativa.
    

---

### 8\. Agendamento da apresentação da V2

-   Foi alinhado realizar uma apresentação sobre a V2 ao pessoal de desenvolvimento.
    
-   Inicialmente houve dúvida entre terça e quarta, e entre presencial e remoto.
    
-   Ao final, o combinado foi:
    
    -   **terça-feira à tarde**;
        
    -   **formato remoto**.
        

---

### 9\. Apresentação de proposta de módulo/cadastro no produto

-   Foi apresentada uma construção visual de módulo, começando por empresas e avançando para contas correntes.
    
-   A ideia central mostrada foi:
    
    -   permitir cadastro de conta corrente com pouca informação inicial;
        
    -   acompanhar a qualidade/completude do cadastro;
        
    -   trabalhar ciclo de vida/status;
        
    -   permitir origem da conta por API, importação ou ação manual;
        
    -   diferenciar descoberta de conta via integração;
        
    -   conduzir o usuário a complementar dados progressivamente.
        

#### 9.1. Conceito de triagem e qualidade do cadastro

-   A proposta inclui indicadores de pendência e triagem para mostrar o que falta complementar em cada entidade, como conta ou banco.
    
-   O objetivo é permitir cadastro rápido no início e melhoria gradual da qualidade dos dados à medida que o uso exigir mais informação.
    

#### 9.2. Banco: catálogo x relacionamento

-   Foi proposta separação entre:
    
    -   um catálogo mais amplo/exaustivo de bancos, mantido pela equipe e eventualmente também pelo cliente;
        
    -   os bancos com os quais o cliente efetivamente se relaciona;
        
    -   complementações locais do relacionamento, como gerente, agência, tipo de serviço etc.
        

#### 9.3. Cadastros secundários / dados de referência

-   Também foi apresentada uma área para cadastros mais simples/secundários, como listas de apoio.
    
-   Foi sugerido que itens como instrumento bancário e documento de origem poderiam entrar nessa categoria.
    
-   Houve cuidado com nomenclatura, pois o termo usado parecia genérico demais e precisaria ser refinado.
    
-   O entendimento consolidado foi:
    
    -   esses itens são listas de apoio;
        
    -   parte deve vir com setup padrão;
        
    -   o cliente pode editar e/ou criar novos valores em certos casos;
        
    -   atributos mais complexos devem continuar ganhando tratamento específico em telas próprias.
        

---

### 10\. Conversa após o encerramento formal: formato da apresentação e percepção interna

-   Após o encerramento principal, houve continuação da conversa sobre o formato remoto da apresentação.
    
-   Foi comentado que o remoto também ajudaria a evitar exposição excessiva e interpretações internas de que estaria havendo conversa “reservada” ou clima de exclusividade em reuniões presenciais em sala separada.
    
-   Foi compartilhada percepção de que, em ocasiões anteriores, abrir demais a audiência de apresentações ligadas a IA/inovação acabou dispersando o foco do público-alvo principal, que era o pessoal de desenvolvimento.
    
-   Também foi mencionado que uma apresentação anterior sobre IA foi vista como marco importante por ter despertado maior interesse em tecnologias como Python.
    
-   A conclusão prática dessa conversa foi manter o foco no público principal e evitar ampliar demais o grupo sem necessidade.
    

---


## Decisões e combinados

-   Finalizar e arredondar a parte de autenticação da V2 até a próxima sexta, com o escopo atualmente especificado.
    
-   Seguir usando solução pronta para autenticação, com Keycloak como referência discutida.
    
-   Manter testes de performance separados da suíte principal automatizada, via benchmark/manual.
    
-   Caminhar para liberação do caso de fluxo de caixa, mesmo com restrições conhecidas em cenários mais extremos de combinação, desde que isso fique claro.
    
-   Adotar reunião **remota** na **terça-feira à tarde** para apresentação da V2.
    
-   Para acesso às bases Oracle 26ai da NEO, usar ferramenta alternativa portável em vez de mexer no cliente Oracle legado das máquinas.
    

---

## Pendências e próximos passos

-   Concluir a parte de autenticação da V2.
    
-   Reexecutar benchmark de performance em ambiente local mais limpo/estável.
    
-   Validar com Cleiton os testes restantes da integração/plugin, incluindo teste com conta real.
    
-   Aguardar definição do Adolfo sobre liberação ao cliente com base na conversa do dia.
    
-   Refinar nomenclatura e escopo dos cadastros secundários/dados de referência na proposta visual.
    
-   Baixar/usar ferramenta adequada para acesso ao Oracle 26.
    
-   Investigar o problema de permissão/configuração da máquina da Tainá.
    
-   Definir quem executará a POC ligada ao ambiente Oracle/Kubernetes.