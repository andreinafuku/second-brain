# Criar Cluster OKE com VCN Customizada

## Introdução

Este guia fornece um passo a passo detalhado para criar um **cluster Oracle Kubernetes Engine (OKE)** com uma **Virtual Cloud Network (VCN) customizada** na Oracle Cloud Infrastructure (OCI).

### Por Que Usar Custom Create Ao Invés de Quick Create?

**Quick Create** (abordagem anterior):
- ✅ Rápido e simples (1-2 cliques)
- ✅ Configuração automática de VCN
- ❌ Sem controle sobre CIDRs
- ❌ Sem controle sobre subnets
- ❌ Não permite evitar conflitos de CIDR

**Custom Create** (abordagem recomendada):
- ✅ Controle total sobre todas as configurações
- ✅ Pode-se escolher CIDRs sem conflito
- ✅ Pode-se otimizar subnets para o caso de uso
- ✅ Melhor para integração com redes existentes
- ⏱️ Leva alguns minutos a mais

### Problema Resolvido

O cluster anterior foi criado com VCN CIDR **10.0.0.0/16**, que é idêntico à VCN **xrt-interno** (que contém o banco de dados). Isso impede a configuração do Local Peering Gateway (LPG).

**Solução:** Criar novo cluster com VCN CIDR **10.2.0.0/16** para eliminar conflito.

## Pré-Requisitos

- ✅ Acesso ao OCI Console (console.oracle.com)
- ✅ Permissões IAM para:
  - `vcn:ManageVcns` - Criar e gerenciar VCNs
  - `container:ManageClusters` - Criar clusters OKE
  - `compute:ManageInstances` - Criar e gerenciar nós
  - `core:ManageSubnets` - Criar e gerenciar subnets
- ✅ Tenancy em `sa-saopaulo-1` (São Paulo)
- ✅ SSH key pair para acesso aos nodes
- ✅ OCI CLI configurado localmente (opcional, para operações avançadas)

## Parte 1: Criar VCN Manualmente

### Passo 1.1: Acessar OCI Console

1. Acesse [console.oracle.com](https://console.oracle.com)
2. Faça login com suas credenciais
3. Selecione a região **São Paulo (sa-saopaulo-1)** no canto superior direito
4. Navegue até **Networking → Virtual Cloud Networks**

### Passo 1.2: Criar VCN Principal

1. Clique em **Create VCN**
2. Preencha os dados:
   - **Name:** `oke-treasury-vcn` (ou outro nome conforme seu padrão)
   - **CIDR Block:** `10.2.0.0/16`
   - **Compartment:** (selecione o compartment apropriado)
   - **DNS Label:** `oketreasury` (sem hífens)

3. Clique em **Create VCN**

> ⏱️ Aguarde alguns segundos até a VCN ser criada.

### Passo 1.3: Criar Internet Gateway

Necessário para acesso externo ao cluster.

1. Após criar a VCN, você será redirecionado para a página de detalhes
2. Na seção **Resources** (esquerda), clique em **Internet Gateways**
3. Clique em **Create Internet Gateway**
4. Preencha:
   - **Name:** `oke-igw`
   - **Enabled:** Marque ✓
5. Clique em **Create Internet Gateway**

### Passo 1.4: Criar Route Table

1. Na seção **Resources** (esquerda), clique em **Route Tables**
2. Clique em **Create Route Table**
3. Preencha:
   - **Name:** `oke-public-route`
   - **Compartment:** (mesmo compartment da VCN)
4. Clique em **Create Route Table**
5. Após criar, configure a rota padrão:
   - Clique na route table criada
   - Clique em **Add Route Rule**
   - Preencha:
     - **Destination CIDR Block:** `0.0.0.0/0`
     - **Target Type:** `Internet Gateway`
     - **Target Internet Gateway:** `oke-igw`
   - Clique em **Add Route Rule**

### Passo 1.5: Criar Security List (Firewall)

1. Na seção **Resources**, clique em **Security Lists**
2. Clique em **Create Security List**
3. Preencha:
   - **Name:** `oke-default-seclist`
   - **Compartment:** (mesmo compartment)
4. Clique em **Create Security List**
5. Após criar, configure as regras necessárias (detalhes na seção "Configurar Security Lists")

### Passo 1.6: Criar Subnets

Você precisa criar 3 subnets com finalidades diferentes:

#### Subnet 1: API Endpoint (Control Plane)

1. Na seção **Resources**, clique em **Subnets**
2. Clique em **Create Subnet**
3. Preencha:
   - **Name:** `oke-api-subnet`
   - **CIDR Block:** `10.2.0.0/28` (14 endereços disponíveis)
   - **Route Table:** `oke-public-route`
   - **Security List:** `oke-default-seclist`
   - **DNS Label:** `okapi`
4. Clique em **Create Subnet**

#### Subnet 2: Worker Nodes

1. Clique em **Create Subnet** (novamente)
2. Preencha:
   - **Name:** `oke-workers-subnet`
   - **CIDR Block:** `10.2.10.0/24` (254 endereços disponíveis)
   - **Route Table:** `oke-public-route`
   - **Security List:** `oke-default-seclist`
   - **DNS Label:** `okeworkers`
3. Clique em **Create Subnet**

#### Subnet 3: Load Balancer (NGINX Ingress)

1. Clique em **Create Subnet** (novamente)
2. Preencha:
   - **Name:** `oke-lb-subnet`
   - **CIDR Block:** `10.2.20.0/24` (254 endereços disponíveis)
   - **Route Table:** `oke-public-route`
   - **Security List:** `oke-default-seclist`
   - **DNS Label:** `okelb`
3. Clique em **Create Subnet**

> ✅ Você agora tem 3 subnets criadas. Todas estão na mesma route table com acesso ao Internet Gateway.

## Parte 2: Configurar Security Lists

As security lists atuam como firewalls em nível de subnet. Você precisa configurar regras de ingress e egress.

### Passo 2.1: Configurar Regras de Ingress

1. Acesse **Virtual Cloud Networks** → `oke-treasury-vcn` → **Security Lists** → `oke-default-seclist`
2. Clique em **Add Ingress Rule** e adicione as seguintes regras:

#### Regra 1: HTTPS (porta 443)
- **Stateless:** Desmarque
- **Direction:** Ingress
- **Source Type:** CIDR
- **Source CIDR:** `0.0.0.0/0`
- **IP Protocol:** TCP
- **Destination Port Range:** `443`
- **Description:** `HTTPS for Ingress`

#### Regra 2: SSH (porta 22) - Apenas de IPs autorizados
- **Stateless:** Desmarque
- **Direction:** Ingress
- **Source Type:** CIDR
- **Source CIDR:** `<SEU_IP_PUBLICO>/32` (ou `0.0.0.0/0` se inseguro)
- **IP Protocol:** TCP
- **Destination Port Range:** `22`
- **Description:** `SSH to nodes`

#### Regra 3: Kubernetes API (porta 6443)
- **Stateless:** Desmarque
- **Direction:** Ingress
- **Source Type:** CIDR
- **Source CIDR:** `0.0.0.0/0`
- **IP Protocol:** TCP
- **Destination Port Range:** `6443`
- **Description:** `Kubernetes API`

#### Regra 4: Pod CIDR Internal (tipo-dependente)
- **Stateless:** Desmarque
- **Direction:** Ingress
- **Source Type:** CIDR
- **Source CIDR:** `10.2.0.0/16` (própria VCN)
- **IP Protocol:** All Protocols
- **Description:** `Internal Pod Traffic`

### Passo 2.2: Configurar Regras de Egress

1. Clique em **Add Egress Rule** e adicione:

#### Regra 1: Todo tráfego para internet
- **Stateless:** Desmarque
- **Direction:** Egress
- **Destination Type:** CIDR
- **Destination CIDR:** `0.0.0.0/0`
- **IP Protocol:** All Protocols
- **Description:** `Allow all outbound traffic`

> ✅ Security lists configuradas. O cluster agora tem acesso à internet e permite tráfego interno.

## Parte 3: Criar Cluster OKE via Custom Create

### Passo 3.1: Acessar Cluster Creation

1. Navegue até **Containers & Artifacting → Kubernetes Clusters (OKE)**
2. Clique em **Create Cluster**
3. Selecione **Custom Cluster** (não Quick Create)

### Passo 3.2: Configurar Cluster Details

Na seção **Cluster Configuration**:

1. **Name:** `oke-treasury-prod` (ou seu padrão)
2. **Kubernetes Version:** Selecione a versão mais recente disponível (ex: v1.28.2)
3. **Visibility:** `Public Endpoints` (acesso público à API)
4. **Compartment:** Mesmo compartment das VCNs

Na seção **Networking**:

5. **Virtual Cloud Network:** Selecione `oke-treasury-vcn`
6. **Kubernetes Service LB Subnet:** `oke-lb-subnet` (10.2.20.0/24)
7. **Pod Communication Subnet:** Deixe a padrão ou use `10.244.0.0/16`

Na seção **Kubernetes Network Config**:

8. **Services CIDR Block:** `10.96.0.0/12` (padrão do Kubernetes)
9. **Pods CIDR Block:** `10.244.0.0/16` (padrão do Kubernetes)

Clique em **Next** ou continue para **Node Pool Configuration**.

### Passo 3.3: Configurar Node Pool

Na seção **Node Pool**:

1. **Node Pool Name:** `treasury-workers`
2. **Version:** Mesmo do cluster (auto-selecionado)
3. **Quantity per Subnet:** `3` (3 nodes, 1 por availability domain)

Na seção **Nodes Configuration**:

4. **Image:** `Oracle Linux 8` (padrão OCI)
5. **Shape:** `VM.Standard.E4.Flex`
   - **OCPUs:** `2`
   - **Memory (GB):** `16`
6. **Node Source (Image):** Latest
7. **Subnet:** `oke-workers-subnet` (10.2.10.0/24)
8. **Assign Public IP Address:** `ENABLE` ✓ (conforme escolha do projeto)

Na seção **Additional Options**:

9. **SSH Key:** Selecione sua SSH public key
   - Se não tiver, gere uma: `ssh-keygen -t rsa -b 4096`
   - Faça upload da public key

10. **Add Labels (opcional):**
    - Key: `workload-type`, Value: `treasury`

Clique em **Next** ou **Create Cluster**.

### Passo 3.4: Revisar e Criar

1. Revise todas as configurações
2. Verifique que a VCN é `oke-treasury-vcn` (10.2.0.0/16)
3. Clique em **Create Cluster**

> ⏱️ Aguarde 10-15 minutos para o cluster ser criado. Você pode acompanhar o progresso no OCI Console.

## Parte 4: Configurar kubectl Local

### Passo 4.1: Baixar Kubeconfig

Após o cluster estar pronto (status "Active"):

```bash
# Obter OCID do cluster
oci ce cluster list \
  --region sa-saopaulo-1 \
  --query "data[0].id" \
  --raw-output

# Armazenar em variável
export CLUSTER_ID="<OCID_DO_CLUSTER>"

# Criar kubeconfig
oci ce cluster create-kubeconfig \
  --cluster-id "$CLUSTER_ID" \
  --file ~/.kube/config \
  --region sa-saopaulo-1 \
  --token-version 2.0.0
```

> ✅ Se o arquivo ~/.kube/config já existe, você pode sobrescrever ou fazer merge.

### Passo 4.2: Validar Conexão

```bash
# Verificar contexto atual
kubectl config current-context

# Listar nodes do cluster
kubectl get nodes -o wide

# Resultado esperado:
# NAME                           STATUS   ROLES    VERSION   INTERNAL-IP   EXTERNAL-IP
# oke-treasury-prod-worker-1     Ready    <none>   v1.28.2   10.2.10.x     <PUBLIC-IP>
# oke-treasury-prod-worker-2     Ready    <none>   v1.28.2   10.2.10.y     <PUBLIC-IP>
# oke-treasury-prod-worker-3     Ready    <none>   v1.28.2   10.2.10.z     <PUBLIC-IP>
```

### Passo 4.3: Validar System Namespaces

```bash
# Listar pods do sistema
kubectl get pods -n kube-system

# Resultado esperado: coredns, etcd, api-server, etc.
```

## Validação do Cluster

### Checklist de Validação

- [ ] Cluster status é "Active" no OCI Console
- [ ] `kubectl get nodes` mostra 3 nodes em status "Ready"
- [ ] `kubectl get pods -n kube-system` mostra pods sistema rodando
- [ ] `kubectl get svc` mostra kubernetes service na default namespace
- [ ] Internet Gateway está habilitado
- [ ] Route table tem rota padrão para 0.0.0.0/0
- [ ] Subnets estão criadas: API (10.2.0.0/28), Workers (10.2.10.0/24), LB (10.2.20.0/24)

### Testes Básicos

```bash
# Test 1: Criar pod de teste
kubectl run nginx --image=nginx:latest

# Test 2: Expor serviço
kubectl expose pod nginx --port=80 --type=NodePort

# Test 3: Obter acesso
kubectl get svc nginx
# Acesse http://<NODE-IP>:<NODE-PORT>

# Test 4: Cleanup
kubectl delete pod nginx
kubectl delete svc nginx
```

## Próximos Passos

✅ Cluster OKE está pronto com VCN customizada (10.2.0.0/16)

**Próximo:** Configurar Local Peering Gateway (LPG) para conectar com banco de dados
- Veja: [`02-configurar-local-peering-gateway.md`](./02-configurar-local-peering-gateway.md)

## Troubleshooting

### Problema: Cluster creation fails

**Solução:**
- Verifique permissões IAM
- Verifique quotas de recursos (compute, storage)
- Revise os logs no OCI Console → Cluster → Work Requests

### Problema: Nodes não aparecem como "Ready"

**Solução:**
```bash
# Verificar status dos nodes
kubectl describe node <NODE-NAME>

# Ver eventos de inicialização
kubectl get events -n kube-system
```

### Problema: kubectl command not found

**Solução:**
```bash
# Instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Problema: kubeconfig permission denied

**Solução:**
```bash
chmod 600 ~/.kube/config
```

## Referências

- [OKE Custom Cluster Creation - Oracle Docs](https://docs.oracle.com/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_Console_to_create_a_Custom_Cluster_with_Explicitly_Defined_Settings.htm)
- [VCN and Subnets - Oracle Docs](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVCNs.htm)
- [kubectl Installation](https://kubernetes.io/docs/tasks/tools/)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
