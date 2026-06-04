---
title: "Anonimizacao de Dados para LLMs em Sistemas Financeiros"
date: 2026-04-13
tags:
  - pesquisa
  - ai
  - arquitetura
  - seguranca
  - status/rascunho
area: arquitetura
---

# Anonimizacao de Dados para LLMs em Sistemas Financeiros

## Contexto

O Intelligence Portal v2 fara uso intensivo de modelos LLM. Existe interesse em utilizar provedores externos como OpenAI, Anthropic e Google, mas isso exige uma estrategia clara de anonimização para dados financeiros sensiveis.

## Registro bruto da analise

O inteligence portal v2 fará uso intensivo de modelos LLM. Temos percebido que o provedor Oracle OCI Generative AI possui desempenho e qualidade inferior aos modelos disponiveis pela OpenAI, Anthropic e Google. A vantagem do uso do Oracle OCI está no fato dos modelos serem estáticos, isto é, são "executados" nos datacenters da Oracle, em que podemos escolher uma regiao do nosso pais, Brasil. Para usar modelos da OpenAI, Anthropic, Google, Gemini, será necessário um plano para que os dados do sistema, que são financeiros, sejam anonimizados antes de serem utilizados pelos modelos. Considerando a hipótese de desenvolvermos um novos sistema (frontend, backend, database, etc), que sugestoes voce tem sobre anonimização? Teremos que considerar este quesito na modelagem do sistema ? (OO e database)?

  

Pensou por 28s

Sim — eu trataria **anonimização/pseudonimização como requisito arquitetural de primeira classe**, não como um detalhe técnico adicionado no fim.

E, para um sistema financeiro com uso intensivo de LLM, eu iria além: **a arquitetura deve ser desenhada assumindo que existe uma fronteira de dados sensíveis antes de qualquer chamada ao modelo**. Isso afeta backend, domínio, banco, observabilidade e até contratos de API.

Hoje, os provedores têm controles corporativos relevantes, mas com diferenças importantes. Na API da OpenAI, os dados enviados não são usados para treino por padrão, e há documentação sobre retenção e controles de dados. A Anthropic informa retenção padrão de 30 dias para API comercial, com opções contratuais como zero data retention em cenários elegíveis. No Google Cloud/Vertex AI, a documentação afirma restrição de treinamento com dados do cliente sem permissão prévia. Já a Oracle disponibiliza OCI Generative AI na região Brazil East (São Paulo), o que reforça a vantagem de residência regional de dados no Brasil. 

Dito isso, mesmo com bons termos contratuais, para dados financeiros eu recomendo partir do princípio de que **o modelo externo só deve receber o mínimo necessário, já desidentificado e com política explícita de exposição**.

## Minha recomendação principal

Estruturem o sistema em **3 zonas de dados**:

1. **Dados identificados**  
    Zona transacional normal: cliente, conta, documento, contrato, e-mail, telefone, chave PIX, favores, histórico etc.
    
2. **Dados protegidos / tokenizados**  
    Dados ainda úteis para correlação interna, mas com substituição de identificadores reais por tokens estáveis. Exemplo: `Cliente 98172`, `Conta_Tok_44A1`, `Contrato_Tok_77`.
    
3. **Dados prontos para LLM**  
    Payload mínimo e sanitizado, contendo apenas:
    
    - contexto funcional necessário
        
    - valores agregados ou mascarados
        
    - identificadores artificiais
        
    - metadados de sensibilidade
        
    - instruções de política sobre o que o modelo não pode inferir nem solicitar
        

Em outras palavras: **não deixar a aplicação “mandar JSON do domínio direto para o LLM”**.

## Sim, isso deve entrar na modelagem OO e de banco

### Na modelagem OO

Eu criaria conceitos explícitos no domínio/aplicação, como:

- `DataClassification`
    
- `SensitivityLevel`
    
- `PiiField`
    
- `LlMAllowedContext`
    
- `RedactionPolicy`
    
- `TokenizationPolicy`
    
- `PromptPayloadBuilder`
    
- `SensitiveContentGuard`
    

Isso evita que o sistema trate anonimização como `string.Replace()` espalhado no código.

Uma boa separação seria:

- **Domínio**: regras sobre o que é sensível e quais dados podem sair do bounded context
    
- **Aplicação**: orquestração de sanitização, montagem do prompt, auditoria e chamada ao provider
    
- **Infraestrutura**: tokenização, criptografia, mascaramento, secrets, KMS/HSM, provider adapters
    

### No banco de dados

Eu modelaria os dados com capacidade de:

- classificar colunas por sensibilidade
    
- separar dados identificáveis de dados operacionais
    
- manter tabelas de mapeamento de token ↔ valor real
    
- registrar trilha de auditoria de cada envio para LLM
    
- definir retenção diferenciada para prompts, outputs e logs
    

Exemplo conceitual:

- `Cliente`
    
- `ContaFinanceira`
    
- `DocumentoFiscal`
    
- `TokenMap`
    
- `PromptAudit`
    
- `PromptPayloadSnapshot`
    
- `LlmInteractionPolicy`
    

## O ponto mais importante: anonimização tem níveis

Nem tudo é “anonimizar” no sentido estrito. Em arquitetura de software, eu separaria assim:

### 1. Mascaramento

Oculta parcialmente o dado.

Exemplo:

- CPF: `***.123.***-**`
    
- Conta: `****6789`
    

Bom para UI e logs. Ruim para muitos casos de LLM, porque ainda pode existir reidentificação.

### 2. Pseudonimização / tokenização

Substitui o valor real por um token reversível internamente.

Exemplo:

- `João da Silva` → `CLI_000184`
    
- `Agência/Conta 1234/98765-0` → `ACC_TKN_73A9`
    

Esse costuma ser o melhor caminho para sistemas corporativos, porque mantém correlação entre eventos sem expor o valor original.

### 3. Anonimização forte

Remove a possibilidade prática de reidentificação.

Exemplo:

- em vez de mandar a transação individual, manda agregados:
    
    - faixa de valor
        
    - categoria de operação
        
    - perfil de risco
        
    - janela temporal aproximada
        

Isso é excelente para analytics, classificação, sumarização e alguns tipos de copiloto, mas pode reduzir precisão em workflows operacionais.

## Estratégia que eu sugiro para o intelligence portal v2

### 1. Classificação de dados desde o início

Criem uma taxonomia simples, por exemplo:

- **Público**
    
- **Interno**
    
- **Confidencial**
    
- **Sensível**
    
- **Altamente sensível/regulado**
    

E classifiquem:

- entidades
    
- campos
    
- eventos
    
- documentos
    
- anexos
    
- logs
    
- embeddings
    
- outputs do LLM
    

Sem isso, a anonimização vira subjetiva.

### 2. Catálogo de campos sensíveis

Mantenham um inventário explícito:

- nome completo
    
- CPF/CNPJ
    
- conta/agência
    
- chave PIX
    
- telefone
    
- e-mail
    
- endereço
    
- número de contrato
    
- número de título
    
- dados de recebíveis
    
- dados bancários de favorecidos
    
- histórico transacional detalhado
    

Cada campo deve ter política:

- pode sair para LLM?
    
- somente mascarado?
    
- somente tokenizado?
    
- nunca pode sair?
    
- pode ir para embedding?
    
- pode aparecer em log?
    

### 3. Gateway de privacidade antes do provider

Toda chamada a OpenAI/Anthropic/Google deveria passar por um componente único, algo como:

`LLM Privacy Gateway`

Responsabilidades:

- detectar campos sensíveis
    
- aplicar mascaramento/tokenização
    
- bloquear payload indevido
    
- anexar metadados de classificação
    
- registrar auditoria
    
- impedir prompt injection que tente pedir dado sensível
    
- controlar qual provider pode receber qual categoria de payload
    

Esse componente é crítico. Eu não deixaria cada squad chamar provider diretamente.

### 4. Prompt com contexto mínimo

Em vez de enviar registros brutos, enviar uma representação semântica reduzida.

Exemplo ruim:

- JSON completo da operação financeira, com cliente, conta, beneficiário e histórico
    

Exemplo bom:

- “Transação de saída classificada como pagamento recorrente, valor na faixa 10k–20k, cliente do segmento corporate, operação com divergência documental, status pendente há 3 dias”
    

O modelo recebe contexto suficiente para raciocinar sem receber identidade real.

### 5. Separar casos de uso por risco

Nem todo uso de LLM exige o mesmo nível de proteção.

Eu dividiria em 3 grupos:

**Baixo risco**

- geração de texto institucional
    
- ajuda ao usuário com conteúdo não sensível
    
- UI assistance
    

**Médio risco**

- sumarização de processos internos
    
- classificação de chamados
    
- extração de informações de documentos previamente sanitizados
    

**Alto risco**

- análise de operações financeiras
    
- suporte a decisão
    
- investigação de anomalias
    
- consultas com contexto de cliente real
    

Para alto risco, eu tenderia a:

- usar OCI/local quando possível
    
- ou LLM externo com payload fortemente pseudonimizado e mínimo
    
- ou ainda arquitetura híbrida
    

## Arquitetura híbrida costuma ser a melhor saída

Pelo que você descreveu, eu não escolheria “um provider para tudo”.

Eu faria algo assim:

- **OCI / Brasil** para casos com maior sensibilidade ou exigência de residência
    
- **OpenAI / Anthropic / Google** para casos em que qualidade e capacidade de raciocínio são decisivas, mas com camada forte de sanitização
    
- **roteamento por política**, não apenas por custo ou qualidade
    

Exemplo:

- sumarização de anomalias financeiras sem identificação → OpenAI / Anthropic
    
- assistência sobre registros altamente sensíveis e investigativos → OCI / ambiente com maior controle regional
    
- classificação documental ou extração de campos → provider escolhido por benchmark real do caso
    

## Isso muda o design do banco?

Sim. Bastante.

Eu consideraria:

### Separação física ou lógica de dados

Por exemplo:

- schema transacional identificado
    
- schema analítico/sanitizado
    
- schema de auditoria e tokenização
    

### Tabelas de tokenização

Um mapeamento seguro, com acesso restrito:

- `TokenId`
    
- `EntityType`
    
- `OriginalValueHash`
    
- `EncryptedOriginalValue`
    
- `TokenDisplay`
    
- `CreatedAt`
    
- `Scope`
    

### Logs e auditoria

Guardar:

- quem solicitou a interação
    
- qual caso de uso
    
- qual provider
    
- qual política aplicada
    
- hash do payload original
    
- payload sanitizado enviado
    
- resposta recebida
    
- decisão de pós-processamento
    

### Retenção

Prompts, outputs e traces não devem seguir a mesma retenção do dado transacional principal.

## Isso muda o design OO?

Também.

Eu evitaria entidades de domínio “ricas demais” vazando diretamente para fora. O ideal é que existam objetos específicos para exposição a LLM, como:

- `TransactionAnalysisContext`
    
- `SanitizedCustomerProfile`
    
- `LlmCaseInput`
    
- `RedactedDocumentView`
    

Ou seja: **o modelo não conversa com a entidade de domínio bruta**. Ele conversa com uma projeção controlada.

Isso ajuda muito a manter:

- encapsulamento
    
- rastreabilidade
    
- compliance
    
- testabilidade
    

## Práticas que eu considero obrigatórias

Para um sistema financeiro, eu colocaria como baseline:

- criptografia em repouso e em trânsito
    
- tokenização para identificadores sensíveis
    
- segregação entre dados identificados e dados enviados ao LLM
    
- logs sem PII
    
- observabilidade com redaction nativa
    
- RBAC forte para acesso a prompts e outputs
    
- aprovação de casos de uso por classificação de risco
    
- testes automatizados de vazamento de PII
    
- DLP/validação de payload antes da chamada
    
- catálogo de prompts aprovados por caso de uso
    
- bloqueio de uso livre de “prompt arbitrário com dados reais”
    

## Armadilhas comuns

As maiores que vejo nesse tipo de iniciativa:

**1. Anonimizar só na borda**  
O backend inteiro continua tratando tudo como texto livre e só mascara no fim. Isso falha rápido.

**2. Esquecer logs**  
Muitas vezes o dado não vaza no provider; vaza em APM, tracing, fila, log de erro, analytics, debug.

**3. Enviar documentos brutos**  
PDFs, extratos, contratos e planilhas geralmente carregam muito mais PII do que o time imagina.

**4. Embeddings sem política**  
Vetores também podem carregar conteúdo sensível. O pipeline de embedding precisa da mesma governança.

**5. Prompt engineering sem governança**  
Não adianta ter política se qualquer endpoint consegue montar prompt com dados brutos.

## Minha sugestão prática de decisão arquitetural

Se vocês estão redesenhando frontend, backend e database, eu seguiria esta diretriz:

### Regra mestra

**Nenhuma integração com LLM pode consumir diretamente entidades transacionais identificadas.**

Sempre passar por:

1. seleção de contexto
    
2. classificação
    
3. redaction/tokenização
    
4. validação DLP
    
5. auditoria
    
6. adapter do provider
    

## Em resumo

Minha resposta objetiva é:

**Sim, vocês devem considerar isso já na modelagem OO e no banco.**  
Não como detalhe, mas como **capacidade transversal da plataforma**.

A melhor abordagem, na sua situação, é:

- modelar classificação de sensibilidade desde o domínio
    
- separar projeções sanitizadas para uso por LLM
    
- criar um privacy gateway central
    
- usar tokenização/pseudonimização como padrão
    
- reservar anonimização forte/agregação para analytics e insights
    
- adotar arquitetura híbrida de providers, guiada por risco do caso de uso
    

Se quiser, eu posso montar um **desenho de arquitetura de referência** para esse cenário, com:

- camadas do sistema
    
- componentes de anonimização
    
- modelagem OO sugerida
    
- tabelas principais do banco
    
- fluxo de chamada para OpenAI/Anthropic/Google/OCI.

## Notas relacionadas

- [[ai-gateway]]
- [[ai-gateway-arquitetura-financeiro]]
