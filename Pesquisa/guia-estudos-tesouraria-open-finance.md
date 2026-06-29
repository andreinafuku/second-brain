---
title: "Guia de Estudos - Tesouraria, Open Finance e AI-First"
date: 2026-06-18
tags:
  - pesquisa
  - arquitetura
  - financeiro
  - status/rascunho
area: financeiro
aliases:
  - Guia de Estudos Tesouraria
---

# 📚 Guia de Estudos — Tesouraria, Open Finance e AI-First

> [!info] Origem
> Guia em formato pergunta-resposta a partir dos temas da [[2026-06-10|conversa com Mota]] sobre o novo sistema de tesouraria. As respostas vão além do que foi dito na reunião, para servir de material de estudo. Ver também o [[2026-06-10-insights-takeaways|compilado de insights]].

## 🔗 Open Finance

**P1. O que é Open Finance e quem o regula no Brasil?**
R: Ecossistema regulado pelo **Banco Central** que padroniza o **compartilhamento de dados financeiros** entre instituições, mediante consentimento do titular. Cobre contas, saldos, extratos, cartões, investimentos e empréstimos. Objetivo: portabilidade de dados ao cliente e padronização da integração (antes cada banco tinha formato próprio). Aprofundar em [[open-finance-brasil]] e [[open-finance]].

**P2. O que é o "consentimento" e por que ele é central?**
R: Autorização explícita do dono da conta para um terceiro acessar seus dados. O usuário é redirecionado à tela do próprio banco (internet banking, às vezes com MFA) para aprovar. Sem consentimento válido não há acesso — e ele tem validade e escopo. Conta conjunta pode exigir mais de um titular; em conta PJ quem opera é a tesouraria.

**P3. Por que usar um agregador (ex.: Pluggy) em vez de se conectar direto ao Banco Central?**
R: Certificar-se direto exige estrutura de segurança pesada, tempo e equipe dedicada. Um agregador já certificado funciona como **intermediário**, com API unificada que absorve a complexidade de cada banco. Trade-off: ganha-se velocidade/custo, mas cria-se dependência de fornecedor — daí isolá-lo (P14). Ver [[pluggy-open-finance-api]] e [[referencia-pluggy]].

## 🏦 Tesouraria & Caixa

**P4. O que a tela de "posição intraday" responde, e o que significam saldo inicial, disponível e liquidez?**
R: Responde "como está o caixa **hoje, agora**", consolidado por empresa→banco→conta. **Saldo inicial** = abertura do dia; **entradas/saídas** = movimentos do dia; **saldo final** = posição corrente; **disponível** = parcela utilizável (ligada aos investimentos resgatáveis); **liquidez** = investimento + saldo (quanto dá para mobilizar).

**P5. O que é cash pooling e qual problema resolve?**
R: **Concentrar/redistribuir saldos entre contas** de um grupo — cobrir negativas com excedente de outras — para minimizar juros de descoberto e saldo ocioso. Resolve o problema do tesoureiro: fechar o dia com o menor número de contas no vermelho. É a **ação** que sucede o **diagnóstico**. Ver [[cash-pooling]].

**P6. O que é "descoberto" e o que é "queima de caixa"?**
R: **Descoberto** = saldo abaixo de zero (juros de cheque especial). **Queima de caixa** (*burn rate*) = saída líquida do dia (saídas − entradas); positiva = consumindo caixa, negativa = acumulando. Ambos viram **regras de alerta** nos cockpits.

## 💹 Investimentos

**P7. O que é o CDI e por que "95% do CDI" aparece como critério de alerta?**
R: O **CDI** é a taxa de referência de juros de curtíssimo prazo no Brasil; rendimentos são expressos como "% do CDI". Alerta de "abaixo de 95% do CDI" sinaliza **risco de rentabilidade**: aplicação rendendo pouco frente ao benchmark, a ser revista.

**P8. O que é risco de concentração e como mitigá-lo?**
R: Exposição excessiva a um único banco/emissor — se ele falha, o impacto é desproporcional. Mitiga-se com **limites de concentração** (ex.: alertar se >35% num emissor) e diversificação. No produto, definir o limite dispara **simulação imediata** das posições afetadas.

**P9. O que são debêntures?**
R: Títulos de dívida emitidos por empresas para captar recursos; o investidor empresta e recebe juros. Aparecem como **operação de investimento** com parcelas/fluxos ao longo do tempo.

## 🖥️ Produto & UX

**P10. O que é um "cockpit" e como difere de uma tela transacional?**
R: **Tela consolidadora de gestão** (KPIs, painéis, regras de alerta) que **não cria dado** — deriva indicadores das tabelas transacionais. A transacional mostra o detalhe registro a registro; o cockpit mostra a visão de alto nível para decisão. Ver [[cockpit]].

**P11. O que é "Today at a Glance" e qual o princípio de design?**
R: "Visão rápida" que reúne KPIs de cada tela num lugar só, cada card remetendo à origem. Princípio: **reduzir carga cognitiva** — entender o dia sem navegar tela a tela. Ver [[today-at-a-glance]].

**P12. Por que exibir "em breve" é melhor que dado mock em produção?**
R: Mock indistinguível do real cria **dívida de credibilidade**: o usuário decide sobre dado falso sem saber. "Em breve" é honesto sobre o estado da funcionalidade — mesmo princípio dos "carimbos de status".

## 🏛️ Arquitetura & Modelagem

**P13. Por que desacoplar "conta corrente" de "empresa" foi boa decisão?**
R: A conta é o dado que o Open Finance **sempre** entrega; a relação empresa↔conta é incerta. Forçar o vínculo na criação quebraria a ingestão. Desacoplar deixa o modelo **absorver a incerteza da fonte** e "nunca parar o processo" (vínculo posterior via status de triagem).

**P14. O que é "isolar o fornecedor" e por que não usar o ID do agregador como identificador interno?**
R: Colocar o provedor atrás de uma fronteira (separar conectividade de transações) para trocá-lo sem refazer o sistema. Usar o **ID do fornecedor como identificador** amarra você a ele; mantenha um ID próprio e guarde o do fornecedor só como campo informativo. Conceito relacionado: [[anti-corruption-layer]]. Ver [[load-bearing]].

**P15. Que princípio explica "reaproveitar a árvore de hierarquia matou o painel de parâmetros"?**
R: **Reuso de um mecanismo bem desenhado**: um componente de seleção serve telas, relatórios e segurança, eliminando uma família de telas de configuração. Menos superfície = menos manutenção e UX consistente. Cuidado: reuso entre contextos com requisitos diferentes (negócio vs segurança) exige separação (P18).

## 🔐 Segurança & Controle de Acesso

**P16. Quais os papéis (RBAC) previstos e o que é RBAC?**
R: **RBAC** (Role-Based Access Control) = acesso por papel, não por usuário. Papéis: **administrador** (governa acessos), **tesoureiro** (operacional/supervisor) e **visualizador** (consulta). O acesso a *dados* vem da árvore de hierarquia (qual nó/empresas o usuário enxerga). Aprofundar em [[auth0-lesson-10-roles-permissions-rbac]].

**P17. O que é maker-checker (dupla autorização) e por que importa em tesouraria?**
R: Padrão de **segregação de funções**: quem **cria** (maker) ≠ quem **aprova** (checker). Em operações sensíveis (concessão de acesso, pagamentos) evita concentração de poder — reduz fraude/erro e atende auditoria. Trade-off: pode virar gargalo se mal calibrado. Ver [[maker-checker]].

**P18. Por que separar a hierarquia de segurança da hierarquia de negócio?**
R: Requisitos opostos: a de **relatório/negócio** quer ser livre e editável; a de **acesso/segurança** precisa ser **estável** (editá-la pode conceder/remover acessos silenciosamente). Misturá-las arrisca que uma mudança de negócio quebre a segurança. Solução: tipo de hierarquia dedicado, sob o Admin.

**P19. O que é MFA e onde entra no fluxo?**
R: **MFA** (Multi-Factor Authentication) exige mais de um fator (senha + token/app). Entra na conexão com bancos via Open Finance — alguns exigem MFA ou código de operador, outros só usuário/senha. Ver [[auth0-lesson-11-security-best-practices]].

## 🤖 Metodologia AI-First

**P20. O que muda ao tratar a IA como "dev sênior" em vez de "dev júnior"?**
R: No modelo júnior, você **desenha a solução** e a IA executa o bruto — o teto é o seu conhecimento. No sênior, você dá **problema + contexto + critérios de aceite** e deixa a IA **propor opções**, que você debate. Aproveita o repertório da IA além do seu ("cada um tem o Claude que merece").

**P21. Por que os "critérios de aceite" são o ponto de apoio do método?**
R: Permitem **avaliar objetivamente** a solução proposta (performance, custo, sem ponto único de falha, P95). Sem critérios, "deixar a IA decidir" deixa de ser delegação informada e vira aceitação cega — o risco oposto do entusiasmo ("não tiro nada daqui").

**P22. O que é P95 (percentil 95) e por que serve de critério?**
R: Valor abaixo do qual caem 95% das observações. "P95 de 200ms" = 95% das requisições respondem em ≤200ms — captura a **cauda** (os 5% piores), bem melhor que a média, que esconde outliers. Por isso é critério de aceite de performance. Ver [[percentil]].

**P23. O que é "ponto único de falha" (SPOF) e por que entra nos critérios?**
R: **SPOF** (Single Point of Failure) = componente cujo colapso derruba o sistema todo (ex.: um único banco de dados). Critério: a solução não pode introduzir um SPOF. Mitiga-se com redundância/replicação.

**P24. Qual o papel dos testes automatizados gerados a partir dos critérios de aceite?**
R: Cada critério vira um teste — tornando "pronto" **verificável e repetível**. Casos custosos (ex.: medir P95) podem ser execução manual, com resultado anexado antes da aprovação da US. É a ponte entre especificação e verificação.

**P25. O que é MCP e por que "contexto atrelado" importa?**
R: **MCP** (Model Context Protocol) é um padrão para conectar modelos de IA a ferramentas/fontes de dados de forma estruturada. "Contexto atrelado" = manter pesquisa, código e dados no mesmo fluxo do agente; sair dele perde contexto e duplica trabalho.

**P26. O que significa, em AI-First, prever sponsor de negócio e dono técnico para um agente?**
R: Se agentes de IA viram "usuários" do sistema, precisam de **governança**: um **sponsor de negócio** (responsável pelo uso) e um **dono técnico** (responsável pelo comportamento/credenciais) antes de receberem autenticação e permissões. É segregação de responsabilidade aplicada a agentes.

## ⚛️ Frontend / API

**P27. Qual a diferença entre fetch e Axios?**
R: **fetch** é a API nativa do navegador (sem dependência, mais leve; exige tratar manualmente JSON, erros HTTP e timeouts). **Axios** é biblioteca ("baterias inclusas": interceptors, JSON automático, cancelamento, melhor erro), ao custo de uma dependência. A escolha por fetch na reunião foi por **leveza**.

**P28. O que é "trocar o mock pela API" no fluxo de implementação?**
R: O protótipo usa **dados simulados (mock)** para validar UI/navegação. A implementação substitui o mock pelas **chamadas de API** reais, mantendo o front-end do protótipo como "fonte da verdade" do comportamento esperado.

## 🧭 Gestão & ERP

**P29. Por que o modelo de acesso se inspira em ERPs e auditoria?**
R: Em ERPs financeiros, **o admin concede acesso** (não se dá amplos poderes a quem opera) e ações sensíveis são **auditadas** — em especial a pagar, a receber e tesouraria. O novo sistema adota isso (concessão pelo Admin + maker-checker) para atender clientes com compliance forte.

**P30. Qual a lição de gestão sobre disponibilidade e delegação?**
R: Profundidade exige **desconexão**: estar 100% disponível impede foco e impede o time de ganhar autonomia ("dar uma canseira no outro lado"). Mas delegar só funciona se a transição for **formalizada e realmente fechada**, não apenas anunciada.

## 🔗 Notas relacionadas
- Reunião: [[2026-06-10]] · [[2026-06-10-resumo-aprofundado]] · [[2026-06-10-analise-critica]] · [[2026-06-10-insights-takeaways]]
- Pesquisa: [[open-finance-brasil]] · [[pluggy-open-finance-api]] · [[conciliacao-bancaria]] · [[auth0-lesson-10-roles-permissions-rbac]]
- Glossário: [[cockpit]] · [[cash-pooling]] · [[maker-checker]] · [[today-at-a-glance]] · [[open-finance]] · [[load-bearing]]
