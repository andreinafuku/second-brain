# Roteiro Otimizado e Detalhado para Deploy no OCI OKE

Este documento apresenta um roteiro completo e otimizado para publicar aplicações no Oracle Kubernetes Engine (OKE), organizado em fases sequenciais com boas práticas e recomendações.

## 📋 Índice

- [Contexto do Projeto](#contexto-do-projeto)
- [Análise do Plano Original](#análise-do-plano-original)
- [Fase 0: Preparação](#fase-0-preparação)
- [Fase 1: Configuração de Acesso ao OCIR](#fase-1-configuração-de-acesso-ao-ocir)
- [Fase 2: Estratégia de Versionamento de Imagens](#fase-2-estratégia-de-versionamento-de-imagens)
- [Fase 3: Geração de Manifestos Kubernetes](#fase-3-geração-de-manifestos-kubernetes)
- [Fase 4: Build e Push das Imagens](#fase-4-build-e-push-das-imagens)
- [Fase 5: Deploy no OKE](#fase-5-deploy-no-oke)
- [Fase 6: Testes e Validação](#fase-6-testes-e-validação)
- [Sugestões de Melhorias e Automações](#sugestões-de-melhorias-e-automações)
- [Ordem de Execução Recomendada](#ordem-de-execução-recomendada)
- [Próximos Passos de Aprendizado](#próximos-passos-de-aprendizado)

---

## 🎯 Contexto do Projeto

### Infraestrutura Atual
- **Cluster OKE:** 2 nodes criados
- **Ambientes:** Desenvolvimento e Homologação
- **Aplicações:** front-end, back-end e back-end IA

### Organização por Namespaces
- **ts-dev:** Ambiente de desenvolvimento
- **ts-hom:** Ambiente de homologação

### Ferramentas
- **Claude Code:** Para geração dos arquivos YAML
- **Docker:** Para build das imagens
- **OCIR:** Oracle Container Registry para armazenamento de imagens
- **kubectl:** Para interação com o cluster Kubernetes

---

## 📊 Análise do Plano Original

### Plano Inicial Proposto

1. Gerar arquivos de definição .yaml kubernetes para front-end, back-end e back-end IA
2. Gerar imagem (docker) do front-end, back-end e back-end IA
3. Configurar, na máquina local, conexão do Oracle Container Registry
4. Fazer upload (push) das imagens para o Oracle Container Registry
5. Configurar no OKE, conexão com Oracle Container Registry
6. Executar os arquivos .yaml no OCI OKE
7. Testar

### Avaliação

O roteiro está **✅ conceitualmente correto**, mas foram identificadas oportunidades de:
- Adicionar etapa de preparação (criação de namespaces)
- Reorganizar ordem de execução (OCIR antes de build)
- Melhorar estratégia de versionamento
- Adicionar automações e boas práticas

---

## 🔧 Fase 0: Preparação

### Importância
Esta fase é **fundamental** e estava ausente no plano original. Sem ela, os deploys falharão.

### 0.1. Criar os Namespaces no Cluster

```bash
kubectl create namespace ts-dev
kubectl create namespace ts-hom
```

**Verificar criação:**
```bash
kubectl get namespaces
```

### 0.2. Configurar Secrets para Acesso ao OCIR

Criar secrets em cada namespace para permitir que o Kubernetes faça pull de imagens privadas:

```bash
# Para ts-dev
kubectl create secret docker-registry ocir-secret \
  --docker-server=<region>.ocir.io \
  --docker-username='<tenancy-namespace>/<oci-username>' \
  --docker-password='<auth-token>' \
  --docker-email=<seu-email> \
  -n ts-dev

# Para ts-hom
kubectl create secret docker-registry ocir-secret \
  --docker-server=<region>.ocir.io \
  --docker-username='<tenancy-namespace>/<oci-username>' \
  --docker-password='<auth-token>' \
  --docker-email=<seu-email> \
  -n ts-hom
```

**Por que isso é importante:**
- Sem os secrets, o Kubernetes não conseguirá baixar imagens do OCIR
- É um pré-requisito para qualquer deploy

---

## 🔐 Fase 1: Configuração de Acesso ao OCIR

> **Nota:** Esta fase foi movida para antes da geração de imagens, pois é mais lógico configurar o acesso antes de começar a fazer push.

### 1.1. Gerar Auth Token no OCI

1. Acessar OCI Console
2. Profile → Auth Tokens
3. Generate Token
4. Salvar token em local seguro

### 1.2. Configurar Credenciais Docker Localmente

```bash
docker login <region>.ocir.io
# Username: <tenancy-namespace>/<oci-username>
# Password: <auth-token>
```

### 1.3. Criar Image Pull Secret no Kubernetes

Já executado na Fase 0.2, mas pode ser atualizado se necessário.

**Documentação detalhada:** Ver arquivo `configurando-acesso-ocir.md`

---

## 🏷️ Fase 2: Estratégia de Versionamento de Imagens

### ❌ Problema com `latest`

A proposta original sugeria usar tag `latest` em desenvolvimento. **NÃO RECOMENDADO!**

**Problemas do `latest`:**
- ❌ Impossível fazer rollback confiável
- ❌ Não se sabe qual versão está rodando
- ❌ Cache pode causar inconsistências
- ❌ Dificulta debug de problemas

### ✅ Estratégia Recomendada de Versionamento

#### Para Desenvolvimento (ts-dev)

**Opção 1: Hash do Git**
```
<nome-app>:dev-<hash-git-curto>
Exemplo: front-end:dev-a3f5b2c
```

**Opção 2: Data + Sequencial**
```
<nome-app>:dev-<YYYYMMDD>-<numero>
Exemplo: back-end:dev-20250130-1
```

**Comando para gerar:**
```bash
# Usando hash do git
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/front-end:dev-$(git rev-parse --short HEAD) .

# Usando data
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/front-end:dev-$(date +%Y%m%d)-1 .
```

#### Para Homologação (ts-hom)

```
<nome-app>:hom-<versao>
Exemplo: back-end:hom-0.1.0
```

#### Para Produção (futuro)

```
<nome-app>:<versao-semantica>
Exemplo: ai-agent:1.2.3
```

### Benefícios desta Estratégia

- ✅ **Rastreabilidade total:** Sabe exatamente qual código está rodando
- ✅ **Rollback simples:** Pode voltar para qualquer versão anterior
- ✅ **Identificação rápida:** Correlação fácil entre imagem e commit
- ✅ **Alinhamento com GitOps:** Facilita automação futura
- ✅ **Depuração eficiente:** Identifica quando bugs foram introduzidos

---

## 📝 Fase 3: Geração de Manifestos Kubernetes

### Estrutura de Diretórios Recomendada

#### Opção 1: Estrutura Simples (Recomendada para Começar)

```
k8s/
├── dev/
│   ├── front-end-deployment.yaml
│   ├── front-end-service.yaml
│   ├── back-end-deployment.yaml
│   ├── back-end-service.yaml
│   ├── ai-agent-deployment.yaml
│   └── ai-agent-service.yaml
└── hom/
    ├── front-end-deployment.yaml
    ├── front-end-service.yaml
    ├── back-end-deployment.yaml
    ├── back-end-service.yaml
    ├── ai-agent-deployment.yaml
    └── ai-agent-service.yaml
```

#### Opção 2: Estrutura Avançada (Para Evolução Futura)

```
front-end/
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── patches/
│       └── hom/
│           ├── kustomization.yaml
│           └── patches/
```

### Componentes Essenciais em Cada YAML

#### 1. Deployment
Define pods, imagens, recursos, health checks

**Elementos obrigatórios:**
- `replicas`: Número de instâncias
- `image`: Caminho completo da imagem no OCIR
- `imagePullSecrets`: Referência ao secret do OCIR
- `resources`: Limites de CPU e memória
- `livenessProbe`: Verifica se o pod está vivo
- `readinessProbe`: Verifica se o pod está pronto para receber tráfego

#### 2. Service
Expõe a aplicação dentro ou fora do cluster

**Tipos:**
- `ClusterIP`: Acesso apenas interno (padrão)
- `LoadBalancer`: Expõe externamente com IP público
- `NodePort`: Expõe em uma porta dos nodes

#### 3. ConfigMap (Opcional mas Recomendado)
Variáveis de ambiente não-sensíveis

#### 4. Secret (Opcional)
Credenciais e dados sensíveis

#### 5. Ingress (Opcional)
Roteamento HTTP/HTTPS externo

### Exemplo de Deployment Completo

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-end
  namespace: ts-dev
  labels:
    app: front-end
    environment: development
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: front-end
    spec:
      imagePullSecrets:
      - name: ocir-secret
      containers:
      - name: front-end
        image: sa-saopaulo-1.ocir.io/<tenancy>/front-end:dev-a3f5b2c
        ports:
        - containerPort: 80
          name: http
        env:
        - name: NODE_ENV
          value: "development"
        - name: API_URL
          value: "http://back-end:8080"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Uso do Claude Code

Para cada aplicação (front-end, back-end, ai-agent):

1. Navegar até a pasta do projeto
2. Executar Claude Code
3. Solicitar geração dos manifestos Kubernetes
4. Revisar e ajustar conforme necessário

---

## 🐳 Fase 4: Build e Push das Imagens

### 4.1. Build Local das Imagens Docker

**Template de comando:**
```bash
docker build -t <region>.ocir.io/<tenancy-namespace>/<repo>/<app>:<tag> ./<app-dir>
```

**Exemplo prático:**
```bash
# front-end com hash do git
cd front-end
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:dev-$(git rev-parse --short HEAD) .

# back-end
cd ../back-end
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/back-end:dev-$(git rev-parse --short HEAD) .

# back-end IA
cd ../ai-agent
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/ai-agent:dev-$(git rev-parse --short HEAD) .
```

### 4.2. Push para OCIR

```bash
docker push sa-saopaulo-1.ocir.io/<tenancy-namespace>/<repo>/front-end:dev-<hash>
docker push sa-saopaulo-1.ocir.io/<tenancy-namespace>/<repo>/back-end:dev-<hash>
docker push sa-saopaulo-1.ocir.io/<tenancy-namespace>/<repo>/ai-agent:dev-<hash>
```

### 4.3. Script de Automação

**`scripts/build-and-push.sh`:**
```bash
#!/bin/bash

# Configurações
REGISTRY="sa-saopaulo-1.ocir.io"
TENANCY="<your-tenancy-namespace>"
REPO="tech-solution"
ENV="dev"

# Função para build e push
build_and_push() {
    APP=$1
    VERSION="${ENV}-$(git rev-parse --short HEAD)"
    IMAGE="${REGISTRY}/${TENANCY}/${REPO}/${APP}:${VERSION}"
    
    echo "🔨 Building ${APP}..."
    docker build -t ${IMAGE} ./${APP}
    
    echo "📤 Pushing ${APP}..."
    docker push ${IMAGE}
    
    echo "✅ ${APP} publicado: ${IMAGE}"
    echo ""
}

# Build e push de todas as aplicações
echo "🚀 Iniciando build e push das imagens..."
echo "==========================================="

build_and_push "front-end"
build_and_push "back-end"
build_and_push "ai-agent"

echo "==========================================="
echo "✅ Todas as imagens foram publicadas!"
```

**Executar:**
```bash
chmod +x scripts/build-and-push.sh
./scripts/build-and-push.sh
```

### 4.4. Verificar Imagens no OCIR

1. Acessar OCI Console
2. Developer Services → Container Registry
3. Verificar se as imagens foram enviadas

---

## 🚀 Fase 5: Deploy no OKE

### 5.1. Verificar Conectividade com Cluster

```bash
# Verificar informações do cluster
kubectl cluster-info

# Listar nodes
kubectl get nodes

# Verificar namespaces
kubectl get namespaces
```

### 5.2. Aplicar Manifestos

**Para ambiente de desenvolvimento:**
```bash
# Aplicar todos os manifestos do diretório dev
kubectl apply -f k8s/dev/ -n ts-dev

# Ou aplicar arquivo por arquivo
kubectl apply -f k8s/dev/front-end-deployment.yaml -n ts-dev
kubectl apply -f k8s/dev/front-end-service.yaml -n ts-dev
kubectl apply -f k8s/dev/back-end-deployment.yaml -n ts-dev
kubectl apply -f k8s/dev/back-end-service.yaml -n ts-dev
kubectl apply -f k8s/dev/ai-agent-deployment.yaml -n ts-dev
kubectl apply -f k8s/dev/ai-agent-service.yaml -n ts-dev
```

**Verificar status:**
```bash
# Listar todos os recursos
kubectl get all -n ts-dev

# Listar pods
kubectl get pods -n ts-dev

# Listar services
kubectl get services -n ts-dev

# Listar deployments
kubectl get deployments -n ts-dev
```

### 5.3. Monitorar Rollout

```bash
# Acompanhar o rollout do front-end
kubectl rollout status deployment/front-end -n ts-dev

# Acompanhar todos os deployments
kubectl rollout status deployment/back-end -n ts-dev
kubectl rollout status deployment/ai-agent -n ts-dev
```

### 5.4. Script de Deploy Automatizado

**`scripts/deploy-dev.sh`:**
```bash
#!/bin/bash

NAMESPACE="ts-dev"
MANIFEST_DIR="k8s/dev"

echo "🚀 Iniciando deploy no ambiente de desenvolvimento..."
echo "=================================================="

# Verificar conectividade
echo "🔍 Verificando conectividade com cluster..."
kubectl cluster-info > /dev/null 2>&1
if [ $? -ne 0 ]; then
    echo "❌ Erro: Não foi possível conectar ao cluster"
    exit 1
fi

# Aplicar manifestos
echo "📦 Aplicando manifestos..."
kubectl apply -f ${MANIFEST_DIR}/ -n ${NAMESPACE}

# Aguardar e verificar status
echo ""
echo "⏳ Aguardando deployments..."
kubectl rollout status deployment/front-end -n ${NAMESPACE}
kubectl rollout status deployment/back-end -n ${NAMESPACE}
kubectl rollout status deployment/ai-agent -n ${NAMESPACE}

# Mostrar status final
echo ""
echo "📊 Status dos recursos:"
kubectl get all -n ${NAMESPACE}

echo ""
echo "✅ Deploy concluído!"
```

---

## ✅ Fase 6: Testes e Validação

### 6.1. Verificar Logs

```bash
# Logs do front-end
kubectl logs -f deployment/front-end -n ts-dev

# Logs do back-end
kubectl logs -f deployment/back-end -n ts-dev

# Logs do ai-agent
kubectl logs -f deployment/ai-agent -n ts-dev

# Logs de um pod específico
kubectl logs <pod-name> -n ts-dev

# Logs de todos os pods de um deployment
kubectl logs -l app=front-end -n ts-dev --all-containers=true
```

### 6.2. Testar Conectividade Entre Serviços

**Executar shell em um pod:**
```bash
kubectl exec -it <pod-name> -n ts-dev -- /bin/sh
```

**Testar comunicação:**
```bash
# De dentro do pod
curl http://back-end:8080/health
curl http://ai-agent:8000/health
```

**Port-forward para teste local:**
```bash
# Encaminhar porta do front-end
kubectl port-forward service/front-end 8080:80 -n ts-dev

# Acessar: http://localhost:8080
```

### 6.3. Validar Endpoints Externos

Se houver LoadBalancer ou Ingress:

```bash
# Obter IP externo
kubectl get services -n ts-dev

# Acessar via navegador ou curl
curl http://<EXTERNAL-IP>
```

### 6.4. Monitoramento de Saúde

```bash
# Verificar eventos
kubectl get events -n ts-dev --sort-by='.lastTimestamp'

# Verificar status dos pods
kubectl get pods -n ts-dev -o wide

# Verificar recursos consumidos
kubectl top pods -n ts-dev
kubectl top nodes
```

---

## 🚀 Sugestões de Melhorias e Automações

### 1. GitOps com Scripts Simples

Estrutura sugerida:
```
scripts/
├── build-all.sh          # Build de todas as imagens
├── push-all.sh           # Push de todas as imagens
├── deploy-dev.sh         # Deploy completo em dev
├── deploy-hom.sh         # Deploy completo em hom
└── rollback.sh           # Rollback para versão anterior
```

### 2. Variáveis de Ambiente Organizadas

**ConfigMap para configurações não-sensíveis:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: back-end-config
  namespace: ts-dev
data:
  NODE_ENV: "development"
  API_URL: "http://ai-agent:8000"
  LOG_LEVEL: "debug"
  DATABASE_HOST: "mysql.ts-dev.svc.cluster.local"
```

**Secret para dados sensíveis:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: back-end-secrets
  namespace: ts-dev
type: Opaque
stringData:
  DATABASE_PASSWORD: "senha-segura"
  API_KEY: "chave-api-secreta"
```

### 3. Health Checks Obrigatórios

Configure `livenessProbe` e `readinessProbe` em todos os Deployments:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

**Por que são importantes:**
- `livenessProbe`: Reinicia pods que travaram
- `readinessProbe`: Remove pods com problemas do load balancing

### 4. Resource Limits

Sempre defina limites de recursos:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**Benefícios:**
- Evita que um pod consuma todos os recursos do node
- Ajuda o scheduler a alocar pods de forma eficiente
- Permite configurar autoscaling

### 5. Estratégia de Deploy Seguro

Use Rolling Update com controle:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Máximo de pods extras durante update
    maxUnavailable: 0  # Garante zero downtime
```

**Vantagens:**
- Zero downtime durante deploys
- Possibilidade de rollback automático se houver falhas
- Atualização gradual e controlada

---

## 📚 Ordem de Execução Recomendada

### Primeira Vez (Setup Completo)

1. ✅ **Criar namespaces**
   - `kubectl create namespace ts-dev`
   - `kubectl create namespace ts-hom`

2. ✅ **Configurar acesso OCIR**
   - Gerar Auth Token no OCI
   - `docker login` local
   - Criar secrets no Kubernetes

3. ✅ **Gerar YAMLs com Claude Code**
   - Começar simples, evoluir depois
   - Um arquivo por vez, testar progressivamente

4. ✅ **Build das imagens**
   - Com versionamento adequado (evitar `latest`)
   - Usar scripts para automação

5. ✅ **Push para OCIR**
   - Verificar no console se imagens estão disponíveis

6. ✅ **Deploy no namespace ts-dev**
   - Aplicar manifestos
   - Monitorar rollout

7. ✅ **Testar e validar**
   - Verificar logs
   - Testar conectividade
   - Validar funcionalidades

8. ✅ **Documentar processo**
   - Registrar comandos usados
   - Documentar problemas encontrados

9. ✅ **Replicar para ts-hom**
   - Quando dev estiver estável e testado

### Deploys Subsequentes

1. ✅ **Fazer alterações no código**
2. ✅ **Commit e push no Git**
3. ✅ **Build nova imagem** (com nova tag/hash)
4. ✅ **Push para OCIR**
5. ✅ **Atualizar YAML** com nova tag de imagem
6. ✅ **Aplicar manifesto** (`kubectl apply`)
7. ✅ **Verificar rollout** (`kubectl rollout status`)
8. ✅ **Testar** novas funcionalidades

---

## 🎓 Próximos Passos de Aprendizado

Quando estiverem confortáveis com o básico:

### Ferramentas de Gerenciamento
- **Kustomize:** Gerenciar variações entre ambientes sem duplicação
- **Helm:** Package manager do Kubernetes, facilita deploys complexos

### CI/CD
- **GitHub Actions:** Automação de build, test e deploy
- **GitLab CI:** Pipeline integrado
- **ArgoCD:** GitOps declarativo

### Observabilidade
- **Prometheus:** Coleta de métricas
- **Grafana:** Dashboards e visualizações
- **Loki:** Agregação de logs
- **Jaeger:** Distributed tracing

### Segurança
- **Pod Security Standards:** Restrições de segurança
- **Network Policies:** Controle de tráfego entre pods
- **Secrets Management:** Vault, External Secrets Operator

### Avançado
- **Horizontal Pod Autoscaler (HPA):** Escala automática baseada em métricas
- **Service Mesh (Istio):** Traffic management, security, observability
- **Cert-Manager:** Gerenciamento automático de certificados SSL/TLS

---

## 📝 Notas Finais

Este roteiro foi otimizado para:
- ✅ Minimizar erros comuns
- ✅ Seguir boas práticas da indústria
- ✅ Facilitar troubleshooting
- ✅ Permitir evolução gradual
- ✅ Servir como base para automação futura

**Lembre-se:**
- Começar simples e evoluir progressivamente
- Documentar tudo que aprender
- Versionar todos os arquivos de configuração
- Testar em desenvolvimento antes de homologação
- Nunca commitar secrets no Git

---

**Última atualização:** Janeiro 2026