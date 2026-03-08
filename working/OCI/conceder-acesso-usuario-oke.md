# Guia: Conceder Acesso de Usuário para Administração OKE

## 📋 Visão Geral

Este guia fornece um procedimento passo a passo para conceder acesso a um usuário da Oracle Cloud Infrastructure (OCI) com permissões para:

- ✅ Criar e administrar clusters OKE (Oracle Kubernetes Engine)
- ✅ Gerenciar VCN (Virtual Cloud Network) e recursos de rede
- ✅ Fazer push/pull de imagens no OCIR (Oracle Cloud Infrastructure Registry)
- ✅ Acessar métricas e logs do cluster
- ✅ Gerenciar Object Storage para backups e artefatos

### Exemplo Prático

Este guia usa como exemplo:
- **Usuário**: `gauss@xtpg.com.br`
- **Compartment**: `xrt-interno`
- **Grupo**: `oke-admins-xrt-interno`
- **Região**: `sa-saopaulo-1` (São Paulo, Brasil)

### Pré-Requisitos

Antes de começar, você precisa de:

- ✅ Acesso ao OCI Console como administrador do tenancy ou do compartment
- ✅ Permissões para criar usuários, grupos e políticas IAM
- ✅ O compartment `xrt-interno` já deve existir
- ✅ Acesso local a um terminal para executar comandos OCI CLI

---

## 🏗️ Arquitetura de Permissões IAM

A hierarquia de permissões no OCI segue este modelo:

```
┌─────────────────────────────────────────────────────────────────┐
│  TENANCY (Sua conta OCI)                                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ IAM POLICIES (Regras de acesso)                          │   │
│  │                                                          │   │
│  │  Política: oke-admin-xrt-interno-policy                  │   │
│  │  ├─ manage cluster-family in compartment xrt-interno     │   │
│  │  ├─ manage virtual-network-family in ...                 │   │
│  │  ├─ manage repos in compartment xrt-interno              │   │
│  │  └─ ... (mais 10+ permissões)                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│           ▲                                                      │
│           │ aplicadas para                                       │
│           │                                                      │
│  ┌────────┴──────────────────────────────────────────────────┐  │
│  │ GROUP: oke-admins-xrt-interno                             │  │
│  │                                                           │  │
│  │  Members:                                                │  │
│  │  └─ gauss@xtpg.com.br ◄────────────┐                    │  │
│  └────────────────────────────────────┬────────────────────┘  │
│                                        │                        │
│                            ┌───────────┘                        │
│                            │                                    │
│  ┌─────────────────────────▼──────────────────────────────┐   │
│  │ USER: gauss@xtpg.com.br                               │   │
│  │ ├─ Email: gauss@xtpg.com.br                           │   │
│  │ ├─ Status: Active                                      │   │
│  │ ├─ API Keys: 1                                         │   │
│  │ ├─ Auth Tokens: 1                                      │   │
│  │ └─ MFA: Habilitado (Recomendado)                       │   │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ COMPARTMENT: xrt-interno (Escopo de acesso)            │   │
│  │ ├─ Clusters OKE                                        │   │
│  │ ├─ VCN e Subnets                                       │   │
│  │ ├─ OCIR Repositories                                   │   │
│  │ ├─ Load Balancers                                      │   │
│  │ ├─ Object Storage Buckets                              │   │
│  │ └─ Métricas e Logs                                     │   │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Fluxo de Permissões:**
1. A **Policy** (regra de acesso) define quem (`group`) pode fazer o quê (`manage`, `use`, `read`) em qual recurso (`cluster-family`, `repos`, etc.)
2. O **Group** (grupo de usuários) agrupa múltiplos usuários com as mesmas permissões
3. O **User** (usuário) é membro do grupo e herda todas as permissões do grupo
4. As permissões se aplicam apenas ao **Compartment** (isolamento de recursos)

---

## 📍 Passo 1: Criar Usuário no OCI

### Via OCI Console

1. Faça login no [OCI Console](https://www.oracle.com/cloud/sign-in.html)
2. Clique no ícone do perfil no canto superior direito → **Identity & Security**
3. Clique em **Domains** (ou **Users** se usando Legacy IAM)
4. Clique em **Create User**
5. Preencha os dados:
   - **First Name**: Gauss
   - **Last Name**: XRT PG
   - **Email**: gauss@xtpg.com.br
   - **Create user in compartment**: Selecione `root` ou o compartment apropriado
6. Clique em **Create**
7. O usuário será criado no status "Provisioning" e depois "Active"
8. Uma notificação será enviada para gauss@xtpg.com.br com instruções de ativação

### Via OCI CLI (Alternativa)

Se preferir criar o usuário via linha de comando:

```bash
# Substitua o compartment-id conforme necessário
oci iam user create \
  --compartment-id ocid1.tenancy.oc1..aaaaaaaaXXXXXXXXXXXX \
  --name gauss.inafuku \
  --email gauss@xtpg.com.br \
  --description "Administrador OKE - Compartment xrt-interno"
```

**Resposta esperada:**
```json
{
  "data": {
    "id": "ocid1.user.oc1..aaaaaaaaXXXXXXXXXXXX",
    "name": "gauss.inafuku",
    "email": "gauss@xtpg.com.br",
    "lifecycle_state": "ACTIVE"
  }
}
```

---

## 👥 Passo 2: Criar Grupo de Usuários

Criar um grupo permite gerenciar múltiplos usuários com as mesmas permissões de forma centralizada.

### Via OCI Console

1. No menu **Identity & Security**, clique em **Groups**
2. Clique em **Create Group**
3. Preencha os dados:
   - **Name**: `oke-admins-xrt-interno`
   - **Description**: `Administradores de clusters OKE no compartment xrt-interno`
4. Clique em **Create**
5. Na tela do grupo criado, clique em **Add User to Group**
6. Selecione `gauss@xtpg.com.br` (ou o nome do usuário criado)
7. Clique em **Add**

### Via OCI CLI

```bash
# Criar grupo
oci iam group create \
  --name oke-admins-xrt-interno \
  --description "Administradores de clusters OKE no compartment xrt-interno"

# Adicionar usuário ao grupo
# Substitua os OCIDs pelos valores reais
oci iam group-membership add \
  --group-id ocid1.group.oc1..aaaaaaaaXXXXXXXXXXXX \
  --user-id ocid1.user.oc1..aaaaaaaaYYYYYYYYYYYY
```

**Benefícios da Abordagem Baseada em Grupos:**

- ✅ **Manutenção centralizada**: Ao adicionar novos usuários ao grupo, herdam automaticamente as mesmas permissões
- ✅ **Auditoria facilitada**: Rastrear permissões por grupo em vez de usuário individual
- ✅ **Remoção rápida**: Remover um usuário do grupo revoga instantaneamente todas as permissões
- ✅ **Escalabilidade**: Suporta dezenas ou centenas de usuários no mesmo grupo

---

## 🔐 Passo 3: Criar Políticas de IAM

As políticas (policies) são o core do modelo de segurança do OCI. Elas definem exatamente quem pode fazer o quê.

### Via OCI Console

1. No menu **Identity & Security**, clique em **Policies**
2. Certifique-se de estar no compartment **root** (seletor no lado esquerdo)
3. Clique em **Create Policy**
4. Preencha os dados:
   - **Name**: `oke-admin-xrt-interno-policy`
   - **Description**: `Política para administração OKE no compartment xrt-interno`
   - **Compartment**: Mantenha como `root` (políticas do tenancy devem estar no root)
5. No campo **Policy Statements**, cole a política completa (veja seção abaixo)
6. Clique em **Create**

### Statements Completos da Política

Cole o seguinte bloco no campo **Policy Statements**:

```
REM ====================================================================================
REM Comentário: Administração OKE - Compartment xrt-interno
REM Grupo: oke-admins-xrt-interno
REM Criado: [data]
REM ====================================================================================

REM ========== Core OKE ==========
REM Gerenciamento completo de clusters OKE, node pools, etc.
Allow group oke-admins-xrt-interno to manage cluster-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to use cluster-node-pools in compartment xrt-interno

REM ========== Compute ==========
REM Gerenciamento de instâncias (worker nodes) e volumes
Allow group oke-admins-xrt-interno to manage instance-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to use volume-family in compartment xrt-interno

REM ========== Networking ==========
REM VCN, subnets, security lists, route tables, load balancers, etc.
Allow group oke-admins-xrt-interno to manage virtual-network-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage load-balancers in compartment xrt-interno

REM ========== Container Registry (OCIR) ==========
REM Push/pull de imagens Docker
Allow group oke-admins-xrt-interno to manage repos in compartment xrt-interno
Allow group oke-admins-xrt-interno to read repos in tenancy

REM ========== Object Storage ==========
REM Backups, logs, artefatos, etc.
Allow group oke-admins-xrt-interno to manage buckets in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage objects in compartment xrt-interno

REM ========== Monitoramento e Logs ==========
REM Acesso a métricas e logs do cluster
Allow group oke-admins-xrt-interno to read metrics in compartment xrt-interno
Allow group oke-admins-xrt-interno to read log-groups in compartment xrt-interno
Allow group oke-admins-xrt-interno to read log-content in compartment xrt-interno

REM ========== Inspeção e Navegação ==========
REM Permissões para navegação no console e acesso a informações de compartments
Allow group oke-admins-xrt-interno to inspect compartments in compartment xrt-interno

REM ========== Cloud Shell e Code Editor ==========
REM Acesso ao Cloud Shell e Code Editor no console OCI
Allow group oke-admins-xrt-interno to use cloud-shell in tenancy
Allow group oke-admins-xrt-interno to use cloud-shell-public-network in tenancy
```

### Via OCI CLI

```bash
# Salvar a política em um arquivo
cat > policy.txt << 'EOF'
Allow group oke-admins-xrt-interno to manage cluster-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to use cluster-node-pools in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage instance-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to use volume-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage virtual-network-family in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage load-balancers in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage repos in compartment xrt-interno
Allow group oke-admins-xrt-interno to read repos in tenancy
Allow group oke-admins-xrt-interno to manage buckets in compartment xrt-interno
Allow group oke-admins-xrt-interno to manage objects in compartment xrt-interno
Allow group oke-admins-xrt-interno to read metrics in compartment xrt-interno
Allow group oke-admins-xrt-interno to read log-groups in compartment xrt-interno
Allow group oke-admins-xrt-interno to read log-content in compartment xrt-interno
Allow group oke-admins-xrt-interno to inspect compartments in compartment xrt-interno
Allow group oke-admins-xrt-interno to use cloud-shell in tenancy
Allow group oke-admins-xrt-interno to use cloud-shell-public-network in tenancy
EOF

# Criar a política
oci iam policy create \
  --compartment-id ocid1.tenancy.oc1..aaaaaaaaXXXXXXXXXXXX \
  --name oke-admin-xrt-interno-policy \
  --statements "$(cat policy.txt)" \
  --description "Política para administração OKE no compartment xrt-interno"
```

### Explicação Detalhada de Cada Permissão

| Permissão | Recurso | Descrição |
|-----------|---------|-----------|
| `manage cluster-family` | OKE Clusters | Criar, editar, deletar clusters OKE; gerenciar node pools |
| `use cluster-node-pools` | Node Pools | Acessar informações de node pools para gerar kubeconfig |
| `manage instance-family` | VMs | Gerenciar instâncias EC2 (worker nodes); reboot, terminate, etc. |
| `use volume-family` | Block Volumes | Criar e gerenciar volumes para persistência de dados |
| `manage virtual-network-family` | VCN, Subnets, NSGs | Criar/editar VCNs, subnets, security lists, route tables |
| `manage load-balancers` | Load Balancers | Criar e gerenciar load balancers (suportam Services do K8s) |
| `manage repos` | OCIR Repositories | Criar repositórios, fazer push de imagens Docker |
| `read repos` | OCIR Repositories (Tenancy) | Fazer pull de imagens públicas de qualquer compartment |
| `manage buckets` | Object Storage | Criar e gerenciar buckets para backups e artefatos |
| `manage objects` | Objects | Upload/download de arquivos em buckets |
| `read metrics` | Monitoring | Visualizar métricas do cluster (CPU, memória, rede) |
| `read log-groups` | Logging | Visualizar logs do cluster e aplicações |
| `read log-content` | Logging | Ler o conteúdo dos logs |
| `inspect compartments` | Compartments | Listar compartments (necessário para navegação no console) |
| `use cloud-shell` | Cloud Shell | Acessar o Cloud Shell e Code Editor via console |
| `use cloud-shell-public-network` | Cloud Shell Network | Permitir acesso à internet a partir do Cloud Shell |

---

## 🐳 Passo 4: Configurar Acesso ao OCIR

O OCIR (Oracle Cloud Infrastructure Registry) é o serviço de container registry do OCI, similar ao Docker Hub.

### 4.1 Gerar Auth Token

Um Auth Token é uma senha temporal para autenticar no OCIR sem usar a senha da conta.

**Via OCI Console:**

1. Clique no ícone do perfil no canto superior direito → **My Profile**
2. Na barra lateral esquerda, clique em **Auth Tokens**
3. Clique em **Generate Token**
4. Digite uma descrição (ex: `OCIR Docker Login - gauss`)
5. Clique em **Generate Token**
6. **⚠️ IMPORTANTE**: Copie o token gerado (aparece apenas uma vez!)
7. Armazene o token em um lugar seguro (gerenciador de senhas, etc.)

**Via OCI CLI:**

```bash
# Gerar um token para o usuário atual
oci iam auth-token create \
  --user-id ocid1.user.oc1..aaaaaaaaXXXXXXXXXXXX \
  --description "OCIR Docker Login"

# Exemplo de resposta:
# {
#   "data": {
#     "token": "YuLJEjvMx0<...muito longo...>dAl5mQ==",
#     "description": "OCIR Docker Login"
#   }
# }
```

### 4.2 Login no OCIR via Docker

Com o Auth Token em mãos, você pode fazer login no OCIR:

```bash
# Substitua:
# - <region-key>: código da região (sa-saopaulo-1 → sasp, us-ashburn-1 → iad, etc.)
# - <tenancy-namespace>: seu namespace do OCI (ex: xyztpg)
# - <username>: gauss.inafuku (ou gauss@xtpg.com.br dependendo da configuração)
# - <auth-token>: o token gerado acima

docker login sasp.ocir.io
# Username: xyztpg/gauss.inafuku
# Password: YuLJEjvMx0<...token...>dAl5mQ==
# Login Succeeded
```

**Encontrar seu region-key:**
| Região | Region Key |
|--------|-----------|
| São Paulo | sasp |
| Ashburn (EUA Leste) | iad |
| Fênix (EUA Oeste) | phx |
| Toronto (Canadá) | yyz |
| Londres (Reino Unido) | lhr |
| Frankfurt (Europa) | fra |
| Singapura | sin |
| Tóquio (Japão) | nrt |

### 4.3 Fazer Push de uma Imagem

Exemplo: Fazer push de uma imagem local para o OCIR

```bash
# 1. Construir uma imagem local
docker build -t myapp:1.0 .

# 2. Tag com o endereço do OCIR
docker tag myapp:1.0 sasp.ocir.io/xyztpg/myapp:1.0

# 3. Fazer push para OCIR
docker push sasp.ocir.io/xyztpg/myapp:1.0

# Resultado esperado:
# The push refers to repository [sasp.ocir.io/xyztpg/myapp]
# 1234567: Pushed
# Latest: digest: sha256:abcd1234... size: 2048
```

### 4.4 Fazer Pull de uma Imagem

```bash
# Pull de uma imagem do OCIR
docker pull sasp.ocir.io/xyztpg/myapp:1.0

# Usar em um container
docker run -d sasp.ocir.io/xyztpg/myapp:1.0
```

### Configurar Credenciais no Kubernetes

Para que o cluster OKE possa fazer pull de imagens privadas do OCIR:

```bash
# Criar um secret no Kubernetes com as credenciais do OCIR
kubectl create secret docker-registry ocir-secret \
  --docker-server=sasp.ocir.io \
  --docker-username=xyztpg/gauss.inafuku \
  --docker-password='YuLJEjvMx0<...token...>dAl5mQ==' \
  --docker-email=gauss@xtpg.com.br \
  -n default

# Usar o secret em um Pod
# No arquivo yaml do Pod, adicione:
# imagePullSecrets:
# - name: ocir-secret
```

---

## ⚙️ Passo 5: Configurar OCI CLI

O OCI CLI (Command Line Interface) é a ferramenta para gerenciar recursos do OCI via linha de comando.

### 5.1 Instalação do OCI CLI

**No macOS (com Homebrew):**
```bash
brew install oci-cli
```

**No Linux (Debian/Ubuntu):**
```bash
curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh | bash
```

**No Windows (com Python):**
```powershell
python -m pip install oci-cli
```

**Ou baixar binário direto:**
```bash
# Acessar https://github.com/oracle/oci-cli/releases
# Baixar o arquivo apropriado para seu SO
```

### 5.2 Configurar Credenciais

O OCI CLI usa API Keys para autenticar. Primeiro, gere uma API Key:

**Via OCI Console:**

1. Clique no seu perfil no canto superior direito → **My Profile**
2. Na barra lateral, clique em **API Keys**
3. Clique em **Add API Key**
4. Escolha **Generate API Key Pair**
5. Clique em **Download Private Key** (salve o arquivo `.pem` em local seguro)
6. Clique em **Add**
7. Na tela seguinte, copie a **Configuration File Preview**

**Criar arquivo ~/.oci/config:**

```bash
# Linux/macOS
mkdir -p ~/.oci
chmod 700 ~/.oci

# Windows
# mkdir %USERPROFILE%\.oci

# Cole o conteúdo do "Configuration File Preview" em ~/.oci/config
# Exemplo:
cat > ~/.oci/config << 'EOF'
[DEFAULT]
user=ocid1.user.oc1..aaaaaaaaXXXXXXXXXXXX
fingerprint=aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88:99
tenancy=ocid1.tenancy.oc1..aaaaaaaaYYYYYYYYYYYY
region=sa-saopaulo-1
key_file=/home/user/.oci/oci_api_key.pem
EOF

# Copiar a chave privada para o local indicado em key_file
cp ~/Downloads/oci_api_key.pem ~/.oci/oci_api_key.pem
chmod 600 ~/.oci/oci_api_key.pem
chmod 600 ~/.oci/config
```

### 5.3 Testar Conectividade

```bash
# Listar regiões (teste básico)
oci iam region list

# Listar compartments
oci iam compartment list

# Listar usuários no tenancy
oci iam user list

# Listar grupos
oci iam group list

# Listar clusters OKE no compartment
oci ce cluster list --compartment-id ocid1.compartment.oc1..aaaaaaaaXXXXXXXXXXXX
```

Se todos os comandos funcionarem, o OCI CLI está corretamente configurado!

---

## ☸️ Passo 6: Configurar kubectl para Acesso ao Cluster OKE

O `kubectl` é a ferramenta para gerenciar clusters Kubernetes. Você precisa de um arquivo `kubeconfig` para conectar ao cluster OKE.

### 6.1 Instalar kubectl

**macOS:**
```bash
brew install kubectl
```

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Windows (PowerShell):**
```powershell
choco install kubernetes-cli
# Ou manualmente:
curl.exe -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"
```

### 6.2 Gerar Kubeconfig

Use o OCI CLI para gerar o arquivo `kubeconfig`:

```bash
# Primeiro, obtenha o CLUSTER-OCID
# Via console OCI > Kubernetes Clusters > Selecionar cluster > Detalhes > Copiar OCID
# Ou via CLI:
CLUSTER_ID=$(oci ce cluster list --compartment-id ocid1.compartment.oc1..XXXX \
  --query 'data[0].id' --raw-output)

# Gerar kubeconfig
oci ce cluster create-kubeconfig \
  --cluster-id $CLUSTER_ID \
  --file ~/.kube/config \
  --region sa-saopaulo-1 \
  --token-version 2.0.0 \
  --kube-endpoint PUBLIC_ENDPOINT

# Resultado:
# New config written to /home/user/.kube/config
```

### 6.3 Testar Conectividade

```bash
# Verificar contexto atual
kubectl config current-context
# Saída esperada: context-abcdef123456

# Listar nodes do cluster
kubectl get nodes
# Saída esperada:
# NAME                           STATUS   ROLES    AGE     VERSION
# 10.2.10.10                     Ready    node     5d      v1.28.0
# 10.2.10.11                     Ready    node     5d      v1.28.0
# 10.2.10.12                     Ready    node     5d      v1.28.0

# Listar pods em todos os namespaces
kubectl get pods --all-namespaces

# Informações do cluster
kubectl cluster-info
kubectl version --short
```

---

## ✅ Passo 7: Validação Completa do Acesso

Após completar todos os passos acima, verifique se o acesso foi configurado corretamente:

### Checklist de Validação

- [ ] **Console OCI**: Faça login com `gauss@xtpg.com.br` (ao menos uma vez)
- [ ] **Compartment**: Consegue visualizar o compartment `xrt-interno` no console
- [ ] **OKE Clusters**: Consegue listar clusters no compartment
  ```bash
  oci ce cluster list --compartment-id ocid1.compartment.oc1..xrt-interno
  ```
- [ ] **OKE Criar**: Consegue visualizar o botão "Create Cluster" no console
- [ ] **VCN**: Consegue visualizar e editar VCNs do compartment
  ```bash
  oci network vcn list --compartment-id ocid1.compartment.oc1..xrt-interno
  ```
- [ ] **OCIR Push**: Consegue fazer push de imagem
  ```bash
  docker login sasp.ocir.io
  # ... (seguir passos 4.2-4.3 acima)
  ```
- [ ] **OCIR Pull**: Consegue fazer pull de imagem
  ```bash
  docker pull sasp.ocir.io/xyztpg/alguma-imagem:tag
  ```
- [ ] **kubectl**: Consegue listar nodes
  ```bash
  kubectl get nodes
  ```
- [ ] **Métricas**: Consegue visualizar métricas do cluster no console
  - OCI Console > Kubernetes Clusters > Selecionar cluster > Monitoring
- [ ] **Logs**: Consegue visualizar logs do cluster
  - OCI Console > Kubernetes Clusters > Selecionar cluster > Logs
- [ ] **Cloud Shell**: Consegue abrir o Cloud Shell no console OCI
  - OCI Console > Ícone do terminal no canto superior direito
- [ ] **Code Editor**: Consegue abrir o Code Editor dentro do Cloud Shell
  - Cloud Shell > Menu ≡ > Code Editor

### Teste de Permissões Detalhado

```bash
# 1. Testar criação de recurso OKE (sem deletar!)
# Tentar criar um novo cluster OKE via console ou CLI
# Deve permitir (mesmo que você cancele depois)

# 2. Testar acesso ao OCIR
oci artifacts container image list --repository-name teste --compartment-id ocid1.compartment.oc1..xrt-interno

# 3. Testar acesso a bucket de object storage
oci os bucket list --compartment-id ocid1.compartment.oc1..xrt-interno

# 4. Testar acesso a métricas
oci monitoring metric-data summarize \
  --namespace oci_kubernetes_engine \
  --query-text 'ClusterCpuUtilization[1h]'

# 5. Testar acesso a logs
oci logging-search search-logs \
  --search-query 'logContent="error"' \
  --time-start 2024-01-01T00:00:00Z
```

---

## 🔧 Troubleshooting

### Erro: "NotAuthorizedOrNotFound" ao listar recursos

**Causa**: Políticas IAM não foram aplicadas ou o usuário não faz parte do grupo correto.

**Solução**:
1. Verifique se `gauss@xtpg.com.br` está no grupo `oke-admins-xrt-interno`
2. Verifique se a política `oke-admin-xrt-interno-policy` existe e está ativa
3. Aguarde ~5 minutos para as mudanças de IAM propagarem
4. Faça logout e login novamente no console OCI

### Erro: "401 Unauthorized" ao fazer login no OCIR

**Causa**: Auth Token expirou, username/password incorretos, ou docker não consegue alcançar sasp.ocir.io.

**Solução**:
```bash
# 1. Verificar conectividade de rede
ping sasp.ocir.io

# 2. Fazer logout e tentar novamente
docker logout sasp.ocir.io
docker login sasp.ocir.io

# 3. Gerar novo Auth Token se o anterior expirou
# (via OCI Console > My Profile > Auth Tokens)

# 4. Verificar formato correto do username
# Deve ser: <tenancy-namespace>/<username>
# Ex: xyztpg/gauss.inafuku
```

### Erro: kubectl não consegue conectar ao cluster

**Causa**: kubeconfig inválido ou não atualizado.

**Solução**:
```bash
# 1. Regenerar kubeconfig
oci ce cluster create-kubeconfig \
  --cluster-id <CLUSTER-OCID> \
  --file ~/.kube/config \
  --region sa-saopaulo-1 \
  --overwrite

# 2. Verificar conectividade de rede
# Certifique-se que consegue alcançar os endpoints públicos do OCI
ping kubernetes.default.svc.cluster.local

# 3. Verificar contexto correto
kubectl config get-contexts
kubectl config use-context <nome-do-contexto>

# 4. Verificar permissões de API Key
# Certifique-se que a API Key ainda é válida:
oci iam api-key list --user-id ocid1.user.oc1..XXX
```

### Erro: "Not authorized to access Code Editor"

**Causa**: Falta permissão de Cloud Shell na política IAM.

**Solução**:
1. Verifique se as seguintes políticas existem no tenancy (root compartment):
   ```
   Allow group oke-admins-xrt-interno to use cloud-shell in tenancy
   Allow group oke-admins-xrt-interno to use cloud-shell-public-network in tenancy
   ```
2. **IMPORTANTE**: Estas políticas devem ser criadas no compartment `root` (tenancy), não no `xrt-interno`
3. Aguarde ~5 minutos para propagação
4. Faça logout e login novamente no console OCI
5. Tente abrir o Cloud Shell novamente (ícone do terminal no header do console)

### Erro: "Policy statement is invalid"

**Causa**: Sintaxe incorreta na política IAM.

**Solução**:
1. Verifique a sintaxe exata: `Allow group <group-name> to <verb> <resource-type> in compartment <compartment-name>`
2. Certifique-se que os nomes do grupo e compartment estão corretos
3. Verifique se há espaços extras ou caracteres especiais
4. Reutilize as statements fornecidas neste guia (já validadas)

### Erro: Docker build/push muito lento ou falhando

**Causa**: Possível issue de conectividade com OCIR ou rate limiting.

**Solução**:
```bash
# 1. Verificar conectividade com OCIR
docker run --rm curlimages/curl curl -v sasp.ocir.io

# 2. Tentar com --no-cache
docker build --no-cache -t myapp:1.0 .

# 3. Fazer push com retry
docker push sasp.ocir.io/xyztpg/myapp:1.0 \
  --retry-max 5 \
  --retry-delay 10

# 4. Verificar logs de rate limiting
# OCI Console > Logging > Logs > Procurar por "rate exceeded"
```

### Como Verificar Logs de Auditoria

O OCI Audit Logs registra todas as ações dos usuários (login, criação de recursos, mudanças de IAM, etc.):

```bash
# 1. Via Console
# OCI Console > Logging > Audit Logs > Filtrar por data/usuário

# 2. Via OCI CLI
oci audit event list \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --query 'data[?principal_id==`ocid1.user.oc1..gauss`]'

# 3. Ver ações específicas de um recurso
oci audit event list \
  --query 'data[?resource_id==`ocid1.cluster.oc1..XXXX`]'
```

---

## 🛡️ Segurança e Boas Práticas

### 1. Princípio do Menor Privilégio

Neste guia, o usuário recebe permissões **apenas** para:
- ✅ Administrar OKE no compartment `xrt-interno` (não em outros compartments)
- ✅ Acessar OCIR, VCN, Object Storage neste compartment
- ✅ Visualizar métricas e logs

Ele **não pode**:
- ❌ Criar novos compartments
- ❌ Gerenciar identidade (criar/deletar usuários/grupos)
- ❌ Acessar recursos fora do compartment `xrt-interno`
- ❌ Deletar a política IAM

Esta é a configuração de segurança ideal.

### 2. Rotação de Auth Tokens (OCIR)

Auth Tokens expiram após certo período. Recomenda-se rotacionar:

```bash
# 1. Gerar novo token via console (My Profile > Auth Tokens > Generate Token)
# 2. Fazer login com o novo token
docker login sasp.ocir.io

# 3. Deletar token antigo no console
# (My Profile > Auth Tokens > Selecionar > Delete)
```

**Periodicidade recomendada**: A cada 90 dias

### 3. Rotação de API Keys (OCI CLI)

De forma similar aos Auth Tokens:

```bash
# 1. Gerar nova API Key no console (My Profile > API Keys > Add API Key)
# 2. Atualizar ~/.oci/config com o novo fingerprint e chave
# 3. Deletar a chave antiga

# Teste
oci iam user list
```

**Periodicidade recomendada**: A cada 90 dias

### 4. Ativação de MFA (Multi-Factor Authentication)

Para usuários com privilégios administrativos, ativar MFA é **altamente recomendado**:

1. Console OCI > My Profile > Devices
2. Clique em **Add Authenticator**
3. Escolha entre:
   - Oracle Mobile Authenticator (recomendado)
   - Google Authenticator
   - Microsoft Authenticator
4. Escaneie o QR code com o app
5. Insira o código gerado no app para confirmar

### 5. Auditoria Periódica

Revise regularmente:

```bash
# Membros do grupo
oci iam group-membership list --group-id ocid1.group.oc1..oke-admins-xrt-interno

# Tokens ativos
oci iam auth-token list --user-id ocid1.user.oc1..gauss

# API Keys ativas
oci iam api-key list --user-id ocid1.user.oc1..gauss

# Mudanças nas políticas
oci audit event list --query 'data[?resource_id==`oke-admin-xrt-interno-policy`]'
```

**Frequência recomendada**: A cada 30 dias

### 6. Uso de Tags para Auditoria

Adicionar tags aos recursos facilita auditoria e controle de custos:

```bash
# Ao criar um cluster, adicionar tags
oci ce cluster create \
  --name meu-cluster \
  --kubernetes-version v1.28.0 \
  --defined-tags '{"financeiro": {"centro-custo": "TI"}, "auditoria": {"criador": "gauss@xtpg.com.br"}}'

# Listar recursos por tag
oci ce cluster list \
  --defined-tag-query '{"financeiro": {"centro-custo": "TI"}}'
```

---

## 📚 Referências

### Documentação Oracle Oficial

- [OCI Identity & Access Management (IAM)](https://docs.oracle.com/iaas/Content/Identity/Concepts/overview.htm) - Visão geral do modelo IAM
- [OKE IAM Policies](https://docs.oracle.com/iaas/Content/ContEng/Concepts/contengpolicyconfig.htm) - Policies específicas para OKE
- [OCIR Authentication](https://docs.oracle.com/iaas/Content/Registry/Tasks/registrypushingimagesusingthedockercli.htm) - Login e uso do OCIR
- [OCI CLI Setup](https://docs.oracle.com/iaas/Content/API/SDKDocs/cliinstall.htm) - Instalação e configuração do OCI CLI
- [Creating a kubeconfig File](https://docs.oracle.com/iaas/Content/ContEng/Tasks/contengdownloadkubeconfigfile.htm) - Gerar kubeconfig
- [OCI Audit Logs](https://docs.oracle.com/iaas/Content/Audit/Concepts/auditoverview.htm) - Rastreamento de ações

### Documentos Relacionados no Projeto

- [Overview OKE](../../ARQUITETURA/oke/README.md) - Arquitetura geral da infraestrutura OKE do projeto
- [Criar Cluster OKE com VCN Customizada](../../ARQUITETURA/oke/01-criar-cluster-oke-vcn-customizada.md) - Procedimento prático de criação de cluster
- [Configurar Local Peering Gateway](../../ARQUITETURA/oke/02-configurar-local-peering-gateway.md) - Conectividade entre VCNs
- [Troubleshooting OKE Deploy](../../FAQ-TROUBLESHOOTING/OKE-DEPLOY-ERRORS.md) - Erros comuns de deploy
- [Conectividade Oracle Database](../../FAQ-TROUBLESHOOTING/oracle-database-connectivity.md) - Conectar cluster ao banco de dados

### Ferramentas Úteis

- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) - Referência rápida do kubectl
- [OCI CLI Reference](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/) - Manual completo do OCI CLI
- [Kubernetes Official Documentation](https://kubernetes.io/docs/) - Documentação do Kubernetes

---

## 📝 Apêndice

### Tabela Completa de Permissões IAM

| Categoria | Permissão | Descrição |
|-----------|-----------|-----------|
| **OKE Core** | `manage cluster-family` | Clusters OKE, node pools, tudo relacionado |
| **OKE Core** | `use cluster-node-pools` | Acesso a node pools para kubeconfig |
| **Compute** | `manage instance-family` | VMs, instâncias, worker nodes |
| **Compute** | `use volume-family` | Block volumes para persistência |
| **Networking** | `manage virtual-network-family` | VCN, subnets, security lists, LPG |
| **Networking** | `manage load-balancers` | LBs para Services K8s |
| **OCIR** | `manage repos` | Criar/deletar repositories, push |
| **OCIR** | `read repos` | Pull de imagens (tenancy-wide) |
| **Object Storage** | `manage buckets` | Criar/deletar buckets |
| **Object Storage** | `manage objects` | Upload/download de arquivos |
| **Monitoring** | `read metrics` | Visualizar métricas |
| **Logging** | `read log-groups` | Listar grupos de logs |
| **Logging** | `read log-content` | Ler conteúdo de logs |
| **Navigation** | `inspect compartments` | Visualizar compartments no console |
| **Cloud Shell** | `use cloud-shell` | Acesso ao Cloud Shell e Code Editor |
| **Cloud Shell** | `use cloud-shell-public-network` | Internet access no Cloud Shell |

### Comandos OCI CLI de Referência Rápida

```bash
# ===== USUÁRIOS =====
oci iam user list
oci iam user get --user-id ocid1.user.oc1..XXX

# ===== GRUPOS =====
oci iam group list
oci iam group-membership list --group-id ocid1.group.oc1..XXX
oci iam group-membership add --group-id ... --user-id ...

# ===== POLÍTICAS =====
oci iam policy list --compartment-id ocid1.tenancy.oc1..XXX
oci iam policy get --policy-id ocid1.policy.oc1..XXX

# ===== CREDENCIAIS =====
oci iam auth-token list --user-id ocid1.user.oc1..XXX
oci iam api-key list --user-id ocid1.user.oc1..XXX

# ===== COMPARTMENTS =====
oci iam compartment list
oci iam compartment get --compartment-id ocid1.compartment.oc1..XXX

# ===== CLUSTERS OKE =====
oci ce cluster list --compartment-id ocid1.compartment.oc1..XXX
oci ce cluster get --cluster-id ocid1.cluster.oc1..XXX
oci ce cluster create-kubeconfig --cluster-id ... --file ...

# ===== OCIR =====
oci artifacts container image list --repository-name XXX
oci artifacts container repository list --compartment-id ...

# ===== AUDIT LOGS =====
oci audit event list --start-time 2024-01-01T00:00:00Z
```

### Glossário de Termos OCI

| Termo | Significado |
|-------|-----------|
| **Tenancy** | Sua conta OCI; isolamento completo de dados e faturamento |
| **Compartment** | Divisão lógica dentro do tenancy; isolamento de recursos e acesso |
| **Policy** | Regra de acesso que define quem (`group`) pode fazer o quê (`verb`) em qual recurso (`resource`) |
| **Group** | Conjunto de usuários com as mesmas permissões |
| **User** | Conta individual de pessoa |
| **OCID** | Oracle Cloud Identifier; ID único para cada recurso (começa com `ocid1.`) |
| **Auth Token** | Token temporário para autenticar no OCIR (similar a senha) |
| **API Key** | Chave privada/pública para autenticar via OCI CLI |
| **VCN** | Virtual Cloud Network; rede virtual isolada |
| **Subnet** | Subrede dentro de uma VCN |
| **Security List** | Firewall para subnets (inbound/outbound rules) |
| **NSG** | Network Security Group; firewall granular para instâncias |
| **Load Balancer** | Balanceador de carga para distribuir tráfego |
| **OCIR** | Oracle Cloud Infrastructure Registry; serviço de container registry (tipo Docker Hub) |
| **OKE** | Oracle Kubernetes Engine; serviço de Kubernetes gerenciado |
| **kubectl** | Ferramenta CLI para gerenciar clusters Kubernetes |
| **kubeconfig** | Arquivo de configuração para autenticar no cluster K8s |

---

## 🆘 Suporte Adicional

Se encontrar problemas não cobertos por este guia:

1. **Consultar documentação Oracle**: [docs.oracle.com](https://docs.oracle.com/)
2. **Abrir chamado no OCI Support**: [OCI Support Portal](https://support.oracle.com/)
3. **Comunidade Oracle**: [Oracle Community](https://community.oracle.com/)
4. **Stack Overflow**: [oci tag on Stack Overflow](https://stackoverflow.com/questions/tagged/oci)

---

**Versão**: 1.0
**Última atualização**: Fevereiro 2026
**Status**: Documentação completa e validada
**Autor**: DeepTreasury Documentation Team
**Exemplos testados em**: OCI Região São Paulo (sa-saopaulo-1), OKE 1.28+, kubectl 1.28+
