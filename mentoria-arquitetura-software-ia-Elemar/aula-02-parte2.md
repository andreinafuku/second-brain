## Aula 2 — continuação da MasterClass Elemar Jr.

### Ideias centrais

-   Arquitetura de software lida menos com escolher tecnologia e mais com **reduzir incerteza**.
    
-   Em sistemas complexos, a melhor solução **não é conhecida antes**; ela aparece após experimentar.
    
-   IA ajuda arquitetura principalmente ao **baratear experimentação**, não só ao acelerar entrega.
    
-   Há uma distinção fundamental entre:
    
    -   **Comportamento estocástico**: pode variar a cada execução
        
    -   **Comportamento determinístico**: deve produzir sempre o mesmo resultado
        

---

## 1\. Revisão conceitual: Cynefin e complexidade

Foi retomado o quadro do **Cynefin**, com quatro tipos de contexto:

### Simples

-   Problemas com resposta conhecida
    
-   Ferramentas e práticas consolidadas resolvem bem
    
-   Há previsibilidade
    

### Complicado

-   Existem várias alternativas possíveis
    
-   Ainda assim, é possível **comparar e decidir antes de implementar**
    
-   A palavra-chave é: **antes**
    
-   Ex.: analisar opções, trade-offs, benchmarks, comparação de soluções
    

### Complexo

-   Existem várias soluções viáveis
    
-   A melhor só aparece **depois de testar no contexto real**
    
-   A palavra-chave é: **depois**
    
-   Exige protótipos, provas de conceito, testes em contexto real
    

### Caótico

-   Nem antes nem depois se consegue prever direito
    
-   O foco tende a ser contenção e resposta rápida
    

### Conclusão aplicada à arquitetura

-   Sistemas de software geralmente estão no campo do **complexo**
    
-   Por isso, arquitetura não pode depender só de análise teórica prévia
    
-   É necessário **experimentar cedo, rápido e barato**
    

---

## 2\. Arquitetura emergente

### O que significa

-   “Arquitetura emergente” não é improvisação
    
-   Não significa decidir sem critério
    
-   Significa reconhecer que, em certos problemas, a solução só vai se revelar ao longo da construção
    

### Princípios associados

-   Experimentar conscientemente
    
-   Fazer experimentos rápidos e baratos
    
-   Voltar atrás quando necessário
    
-   Manter opções abertas por mais tempo
    
-   Registrar critérios e decisões
    
-   Revisar decisões quando surgirem novas evidências
    

### Último momento responsável

-   Algumas decisões não devem ser tomadas cedo demais
    
-   A ideia é adiar até o ponto em que ainda é seguro esperar
    
-   Isso evita fechamento prematuro em alternativas que podem se mostrar ruins depois
    

### Nova evidência

-   Se surgir nova tecnologia, nova restrição ou nova oportunidade:
    
    -   não ignorar automaticamente
        
    -   avaliar impacto
        
    -   entender custo de adoção
        
    -   registrar que a evidência foi considerada
        

---

## 3\. Consciência situacional

-   Algumas coisas só ficam claras **com a solução rodando**
    
-   Há conhecimento que não aparece no planejamento abstrato
    
-   Só se percebe no uso, no teste, na operação, no contexto real
    
-   Isso vale para software, mas também para exemplos do dia a dia:
    
    -   fazer bolo
        
    -   praticar esporte
        
    -   testar interface
        
-   A ideia é desenvolver percepção da situação real, não só da hipótese teórica
    

### Frase-chave

-   Arquitetura não é “adivinhar o futuro”
    
-   É **reduzir incerteza por meio de experimentação**
    

---

## 4\. Como a IA ajuda arquitetura

### Tese principal

A IA não é valiosa só porque “faz mais rápido”.

O ganho real está em:

-   validar hipóteses mais rápido
    
-   gerar alternativas mais barato
    
-   diminuir o custo de experimentar
    
-   acelerar aprendizado
    

### Exemplo de interface

-   O valor não está apenas em gerar uma interface rapidamente
    
-   O valor está em testar se aquela interface **é adequada**
    
-   Ou seja: IA acelera validação, não apenas produção
    

### Consequência para arquitetos

-   O papel do arquiteto é validar hipóteses com mais velocidade
    
-   A IA entra como instrumento para:
    
    -   prototipação
        
    -   experimentação
        
    -   documentação
        
    -   exploração de alternativas
        

---

## 5\. A preocupação central ao usar IA para código

### Pergunta principal

-   Existe contexto suficiente para a IA fazer um bom trabalho?

Essa foi colocada como a grande preocupação de engenharia:

-   se o contexto está ruim, a saída tende a ser ruim
    
-   a solução passa por estruturar melhor o conhecimento do projeto
    

### Consequência prática

-   melhorar documentação
    
-   explicitar convenções
    
-   registrar padrões
    
-   reduzir dependência de tradição oral
    

---

## 6\. Memória de procedimento e arquivos de regra

Foi apresentado um conjunto de práticas para orientar LLMs em desenvolvimento.

### Problema

Modelos são estocásticos:

-   ora escrevem de um jeito
    
-   ora escrevem de outro
    
-   isso cria variabilidade indesejada
    

### Solução

Dar **prescrições explícitas** sobre como o código deve ser escrito.

### Arquivos de regra

-   Em geral, são arquivos Markdown com instruções para a LLM
    
-   Descrevem como agir em determinado contexto
    
-   Podem especificar:
    
    -   padrão de repositório
        
    -   convenção de nomenclatura
        
    -   estrutura esperada
        
    -   o que fazer
        
    -   o que não fazer
        

### Níveis de regra

-   Regra da organização
    
-   Regra do projeto
    
-   Regra por escopo/pasta/contexto
    

### Funções das regras

1.  Ajudar a IA a **escrever**
    
2.  Ajudar a IA a **revisar**
    

### Insight importante

Essas regras são, na prática, a documentação que já deveria existir para humanos:

-   onboarding
    
-   padrões do time
    
-   convenções arquiteturais
    
-   critérios de revisão
    

Ou seja:

-   o que antes era passado por tradição oral
    
-   agora pode ser formalizado e reaproveitado pela IA
    

---

## 7\. Playbooks / procedures / automações

Além de regras, foi discutida a ideia de **procedimentos** para execução de tarefas.

### Objetivo

Quando uma atividade é repetível e previsível, a IA não deve “improvisar” sua execução.

### Estrutura

-   sequência definida de passos
    
-   eventualmente combinada com scripts
    
-   comportamento mais controlado
    

### Ideia central

Para operações operacionais, o ideal é:

-   usar a LLM para planejar ou montar o script
    
-   usar o script para executar
    

Isso barateia e torna o processo mais previsível.

---

## 8\. Estocástico vs determinístico

Esse foi um dos temas mais fortes da aula.

### Estocástico

-   O resultado pode variar
    
-   Mesmo com a mesma pergunta, as respostas podem vir diferentes
    
-   Isso não significa necessariamente erro
    
-   Significa variabilidade inerente ao modelo
    

### Determinístico

-   Mesmo input → mesmo output
    
-   Adequado quando o comportamento desejado é exato, repetível e previsível
    

### Como mitigar variabilidade

-   prompts mais detalhados
    
-   regras mais claras
    
-   formatos mais rígidos
    
-   validação da saída
    
-   temperatura mais baixa
    
-   uso de scripts e ferramentas determinísticas sempre que possível
    

### Formulação forte da aula

Sempre que possível:

-   substituir comportamento estocástico por determinístico

Porque isso torna o sistema:

-   mais previsível
    
-   mais barato
    
-   mais confiável
    
-   mais eficiente
    

---

## 9\. Exemplo marcante: mover arquivos com IA

Foi relatado um caso prático:

-   durante uma refatoração pesada, com renomeações e movimentações de arquivos
    
-   a ferramenta não “moveu” os arquivos
    
-   ela **reescreveu** os arquivos em novos lugares
    

### Consequências

-   alto consumo de créditos/tokens
    
-   custo desnecessário
    
-   comportamento ineficiente
    

### Lição

Pedir para uma LLM executar algo determinístico como “mover arquivo” pode ser inadequado.

### Melhor abordagem

-   usar script para mover arquivos
    
-   usar LLM para gerar ou orientar o script
    
-   não deixar a LLM executar diretamente tarefas que exigem precisão operacional
    

---

## 10\. Uso de agentes, scripts e ferramentas auxiliares

Foi discutido que modelos mais novos têm mostrado tendência a:

-   escrever scripts
    
-   executar scripts
    
-   delegar tarefas mais determinísticas a mecanismos apropriados
    

Isso foi visto como positivo porque:

-   reduz custo de GPU/processamento
    
-   melhora previsibilidade
    
-   separa melhor os tipos de trabalho
    

### Regra prática

-   Parte estocástica: usar LLM
    
-   Parte determinística: usar script, lint, compilador, verificador, parser, análise estática etc.
    

---

## 11\. Onde usar LLM e onde não usar

### Não faz sentido usar LLM para:

-   validar CPF
    
-   checar se um valor é inteiro
    
-   fazer análise sintática que compilador já faz
    
-   validar estrutura que ferramenta determinística já cobre
    
-   consultar informações estruturadas que poderiam vir de busca/indexação/script
    

### Faz sentido usar LLM para:

-   análise textual
    
-   interpretação
    
-   geração de hipóteses
    
-   elaboração de comentários
    
-   análise qualitativa
    
-   síntese
    
-   apoio à revisão mais subjetiva
    

### Critério

Não pedir para LLM fazer análise que uma ferramenta determinística já consegue entregar melhor.

---

## 12\. Ferramentas de análise estática, compiladores e LSP

Foi defendido o uso mais intenso de:

-   análise estática
    
-   compiladores
    
-   LSPs
    
-   serviços de linguagem
    
-   mecanismos estruturados de consulta a código
    

### Motivo

Essas ferramentas:

-   já conhecem a estrutura do código
    
-   respondem com mais precisão
    
-   custam menos
    
-   são mais adequadas para tarefas estruturais
    

### Exemplo citado

-   Em vez de mandar contexto enorme do código para a LLM interpretar:
    
    -   usar o serviço da linguagem/compilador para responder questões estruturais
-   Ex.: referência de símbolo, validade de tipo, uso de classe, acoplamento, estrutura do código
    

### Ideia de fundo

Não usar LLM como se ela fosse compilador.

---

## 13\. Memória, contexto e a frase-chave sobre LLM

### Frase marcante

-   LLMs são gênios com amnésia

### Implicação

Sem contexto e memória:

-   o modelo perde continuidade
    
-   responde pior
    
-   repete trabalho
    
-   consome mais
    

### Solução

Criar mecanismos externos de memória:

-   memória episódica
    
-   memória semântica
    
-   documentos estruturados
    
-   banco de dados pesquisável
    
-   relações entre pessoas/entidades
    
-   busca ao invés de contexto bruto gigante
    

### Exemplo dado

Em vez de abrir centenas ou milhares de arquivos texto:

-   fazer busca estruturada
    
-   recuperar registros relevantes
    
-   passar apenas o necessário para a conversa
    

### Moral

Boa engenharia de contexto é parte essencial do uso sério de IA.

---

## 14\. Planejamento antes de execução

Foi comentado que pedir para a IA fazer primeiro um **plano de execução** é uma boa prática.

### Benefícios

-   permite revisar a abordagem antes da execução
    
-   ajuda a corrigir rumo cedo
    
-   melhora qualidade do resultado
    
-   reduz retrabalho
    
-   permite quebrar tarefas grandes em menores
    

### Justificativa conceitual

-   tarefas menores tendem a ser executadas melhor
    
-   fracionar trabalho melhora aproveitamento da “força” do modelo
    
-   também reduz ida e volta desnecessária
    

### Evolução desejável

-   criar instruções para que a própria IA planeje melhor desde o início
    
-   reduzir correções manuais no plano
    

---

## 15\. Validação da saída da IA

Mesmo quando o prompt é bem definido, ainda pode haver variação.

### Portanto

Deve existir um mecanismo que verifique se a saída:

-   aderiu ao formato pedido
    
-   respeitou a estrutura esperada
    
-   está válida
    

### Exemplos de validação

-   parser de Markdown/JSON
    
-   lint
    
-   verificação sintática
    
-   schema
    
-   checks automatizados
    

### Ideia principal

Saída estocástica deve ser seguida de **verificação determinística**.

---

## 16\. Temperatura

Foi mencionado o uso da variável de **temperatura**.

### Efeito

-   temperatura alta → mais variação/criatividade
    
-   temperatura baixa → menos variação, maior previsibilidade
    

### Observação importante

-   temperatura menor não transforma o modelo em determinístico
    
-   apenas reduz a variabilidade
    

---

## 17\. Regras, documentação e onboarding

Um ponto importante da discussão:

-   arquivos de regra equivalem, em boa parte, à documentação que o time já deveria manter

### Isso inclui

-   estilo de código
    
-   padrões de projeto
    
-   convenções de nomenclatura
    
-   decisões de arquitetura
    
-   práticas proibidas
    
-   critérios de revisão
    

### Mudança de mentalidade

Antes:

-   muito conhecimento ficava implícito
    
-   dependia de revisão manual e tradição oral
    

Agora:

-   isso pode ser explicitado para humanos e IA

---

## 18\. Não existe padrão universal estável ainda

Houve discussão sobre unificar padrões entre ferramentas.

### Resposta geral da aula

-   ainda estamos em um período de evolução rápida
    
-   não existe padrão universal maduro que resolva tudo
    
-   tentar abstrair cedo demais pode limitar recursos específicos de cada ferramenta
    

### Recomendação implícita

-   dominar os fundamentos
    
-   adaptar a aplicação prática a cada ferramenta
    
-   aproveitar particularidades quando fizer sentido
    

### Analogia usada

Como no passado com SQL entre bancos diferentes:

-   houve um longo período de incompatibilidades
    
-   padronização veio só depois de muito tempo
    

---

## 19\. Pensar além do formato humano de documentação

Esse foi um ponto bem interessante do fechamento.

### Problema levantado

-   muita documentação é gerada, mas humanos não leem
    
-   às vezes a documentação está servindo mais para IA do que para pessoas
    

### Pergunta importante

Se o principal leitor for IA:

-   o formato ideal ainda é Markdown/HTML tradicional?
    
-   ou deveria ser algo mais estruturado e econômico?
    

### Exemplos sugeridos

-   JSON estruturado
    
-   formatos mais “parseáveis”
    
-   estruturas mais baratas em tokens
    
-   documentos feitos para consumo de máquina
    

### Insight

Nem toda documentação do futuro precisa ser pensada para leitura humana.

---

## 20\. Heurística prática: 10-80-10

Foi reforçada uma distribuição desejável do trabalho:

### Modelo 10-80-10

-   10% início humano
    
-   80% execução assistida/automatizada
    
-   10% revisão humana final
    

### Uso

-   boa heurística para avaliar se o uso da IA está saudável
    
-   se a distribuição estiver muito fora disso, pode haver uso ineficiente
    

---

## 21\. Exemplos e analogias usados na aula

### Maquete do arquiteto

-   Serve para experimentar antes de construir de fato
    
-   Analogia com protótipos e PoCs
    

### Prova de múltipla escolha vs dissertativa

-   Múltipla escolha:
    
    -   correção mais determinística
        
    -   barata
        
    -   escalável
        
-   Dissertativa:
    
    -   exige interpretação humana
        
    -   mais custosa
        
    -   mais variável
        

### Uso didático da analogia

-   explicar a diferença entre determinístico e estocástico

---

## 22\. Frases e ideias que valem revisão futura

-   Arquitetura não é escolher tecnologia; é lidar com incerteza
    
-   Sistemas de software são complexos
    
-   A melhor solução muitas vezes só aparece depois
    
-   Experimente cedo, rápido e barato
    
-   Arquitetura emergente não é improvisação
    
-   IA reduz o custo da experimentação
    
-   Existe contexto suficiente para a IA fazer um bom trabalho?
    
-   LLMs são gênios com amnésia
    
-   Sempre que possível, substitua estocástico por determinístico
    
-   Onde cabe script, coloque script
    
-   Onde cabe lint, coloque lint
    
-   Saída estocástica deve ser validada de forma determinística
    
-   Nem toda documentação do futuro será para humanos
    

---

## 23\. Resumo executivo da segunda parte

-   A aula conectou arquitetura de software com teoria da complexidade e uso prático de IA.
    
-   O ponto central foi que IA é útil quando **reduz custo de aprendizado e experimentação**.
    
-   Para uso profissional de IA em engenharia, é essencial separar:
    
    -   o que é subjetivo, exploratório e variável
        
    -   do que é repetível, verificável e automatizável
        
-   O melhor uso da IA não é deixá-la fazer tudo, mas **encaixá-la corretamente dentro de um sistema bem projetado**.
    
-   O futuro da engenharia com IA passa por:
    
    -   regras explícitas
        
    -   memória e contexto estruturados
        
    -   validação determinística
        
    -   uso inteligente de scripts, compiladores, LSPs e análise estática
        
    -   documentação pensada também para consumo por máquina
        

Se quiser, eu também posso transformar isso em um dos formatos abaixo:

-   **resumo enxuto de 1 página**
    
-   **flashcards de estudo**
    
-   **mapa mental em tópicos**
    
-   **anotações em Markdown limpo para exportar**