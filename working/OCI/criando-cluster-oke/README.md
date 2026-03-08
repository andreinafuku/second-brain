# Documentação: Cluster OKE com VCN Customizada

## Visão Geral

Este conjunto de documentação fornece um guia completo para criar e configurar um **cluster Oracle Kubernetes Engine (OKE)** com uma **VCN (Virtual Cloud Network) customizada** na Oracle Cloud Infrastructure (OCI).

### Contexto: Por Que Uma VCN Customizada?

O cluster OKE inicial foi criado via **Quick Create** com VCN automática usando CIDR **10.0.0.0/16**. O banco de dados Oracle existente está na VCN **xrt-interno**, também com CIDR **10.0.0.0/16**.

**Problema:** Quando duas VCNs possuem CIDRs idênticos, é impossível estabelecer comunicação entre elas via Local Peering Gateway (LPG).

**Solução:** Criar uma nova VCN com CIDR **10.2.0.0/16** (sem conflito) para o novo cluster OKE e configurar o LPG para conectividade.

## Documentação Disponível

### Ordem Recomendada de Leitura

#### Setup Básico (Obrigatório)

1. **[`01-criar-cluster-oke-vcn-customizada.md`](./01-criar-cluster-oke-vcn-customizada.md)** ⭐
   - Criação de VCN customizada (10.2.0.0/16)
   - Configuração de subnets (API, Workers, Load Balancer)
   - Setup de cluster OKE via Custom Create
   - Validação e testes do cluster

2. **[`02-configurar-local-peering-gateway.md`](./02-configurar-local-peering-gateway.md)** ⭐
   - Criação de Local Peering Gateways
   - Configuração de rotas entre VCNs
   - Configuração de security lists para conectividade
   - Testes de conectividade com banco de dados

#### Componentes Adicionais (Recomendado)

3. **[`03-configurar-ingress-nginx.md`](./03-configurar-ingress-nginx.md)**
   - Instalação do NGINX Ingress Controller
   - Configuração de Ingress para DeepTreasury
   - Roteamento HTTP/HTTPS profissional

4. **[`04-configurar-cert-manager.md`](./04-configurar-cert-manager.md)**
   - Automação de certificados TLS
   - Integração com Let's Encrypt
   - Renovação automática de certificados

5. **[`05-configurar-monitoring.md`](./05-configurar-monitoring.md)**
   - Stack de monitoring (Prometheus + Grafana)
   - Métricas do cluster e aplicação
   - Alertas e observabilidade

#### Migração da Aplicação

6. **[`06-migrar-aplicacao.md`](./06-migrar-aplicacao.md)**
   - Preparação e planejamento de migração
   - Deploy no novo cluster
   - Estratégias de cutover
   - Rollback em caso de necessidade

#### Referência Técnica

7. **[`07-diagrama-arquitetura-rede.md`](./07-diagrama-arquitetura-rede.md)**
   - Diagrama completo da topologia de rede
   - Endereçamento IP e subnets
   - Fluxo de tráfego
   - Security lists e NSGs

## Arquitetura de Rede Proposta

```
┌─────────────────────────────────────────────────────────┐
│        VCN OKE (10.2.0.0/16)                            │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Subnet API Endpoint (10.2.0.0/28) - Public       │   │
│  │ • Kubernetes API control plane                   │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Subnet Workers (10.2.10.0/24) - Public           │   │
│  │ • 3 worker nodes com IPs públicos                │   │
│  │ • DeepTreasury application pods                  │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Subnet Load Balancer (10.2.20.0/24) - Public     │   │
│  │ • NGINX Ingress Controller                       │   │
│  │ • Service LoadBalancer                           │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                           │
                  Local Peering Gateway
                           │
┌─────────────────────────────────────────────────────────┐
│   VCN xrt-interno (10.0.0.0/16) - EXISTENTE           │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Subnet Database (164.152.38.0/24) - Private      │   │
│  │ • Oracle Database: 164.152.38.179:1521           │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Outras subnets...                                │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## Quick Reference: Comandos Essenciais

### Configuração de kubectl

```bash
# Baixar kubeconfig
oci ce cluster create-kubeconfig \
  --cluster-id <CLUSTER-OCID> \
  --file ~/.kube/config \
  --region sa-saopaulo-1 \
  --token-version 2.0.0

# Verificar contexto
kubectl config current-context

# Verificar nodes
kubectl get nodes -o wide
```

### Testes de Conectividade

```bash
# Pod de teste para verificar conectividade ao banco
kubectl run -it --rm debug --image=busybox --restart=Never -- sh

# Dentro do pod, testar conexão TCP
nc -zv 164.152.38.179 1521

# Ou com telnet (se disponível)
telnet 164.152.38.179 1521
```

### Helm (Instalação de componentes)

```bash
# Instalar NGINX Ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# Instalar Cert-Manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true

# Instalar kube-prometheus-stack (Monitoring)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

## Especificações Técnicas

### VCN e Subnets

| Recurso | CIDR / Range | Descrição |
|---------|-------------|-----------|
| VCN OKE | 10.2.0.0/16 | Rede principal do cluster |
| Subnet API | 10.2.0.0/28 | Kubernetes API control plane |
| Subnet Workers | 10.2.10.0/24 | Worker nodes (3 unidades) |
| Subnet LB | 10.2.20.0/24 | NGINX Ingress LoadBalancer |
| **VCN xrt-interno** | **10.0.0.0/16** | Rede existente (banco de dados) |
| **Subnet Database** | **164.152.38.0/24** | Oracle Database |
| **Database IP** | **164.152.38.179:1521** | Oracle Database endpoint |

### Cluster OKE

| Configuração | Valor |
|--------------|-------|
| Kubernetes Version | Latest Stable (1.28+) |
| Cluster Type | Enhanced (managed nodes) |
| Node Pool | treasury-workers |
| Quantidade de Nodes | 3 (alta disponibilidade) |
| Node Shape | VM.Standard.E4.Flex (2 OCPU, 16GB RAM) |
| Node Subnet | 10.2.10.0/24 (public) |
| Total de Recursos | 6 OCPUs, 48GB RAM |

### Componentes Kubernetes

| Componente | Namespace | Descrição |
|-----------|-----------|-----------|
| NGINX Ingress Controller | ingress-nginx | Roteamento HTTP/HTTPS |
| Cert-Manager | cert-manager | Automação de certificados TLS |
| Prometheus | monitoring | Coleta de métricas |
| Grafana | monitoring | Visualização de métricas |
| DeepTreasury App | ts-dev | Aplicação principal |

## Pré-Requisitos

Antes de iniciar, certifique-se de ter:

- ✅ Acesso ao OCI Console (Oracle Cloud Infrastructure)
- ✅ Permissões IAM para:
  - Criar VCNs, subnets e gateways
  - Criar clusters OKE e node pools
  - Gerenciar security lists e route tables
- ✅ OCI CLI configurado localmente (`oci configure`)
- ✅ kubectl instalado (`kubectl version --client`)
- ✅ Helm instalado (recomendado para componentes adicionais)
- ✅ SSH key pair para acesso aos nodes

## Tempo Estimado

| Etapa | Tempo |
|-------|-------|
| Criar VCN e subnets | 5-10 minutos |
| Criar cluster OKE | 10-15 minutos |
| Configurar LPG | 10-15 minutos |
| Instalar NGINX Ingress | 5 minutos |
| Instalar Cert-Manager | 5 minutos |
| Instalar Monitoring | 10 minutos |
| **Total** | **~1-2 horas** |

## Segurança

### Considerações Importantes

1. **Subnets Workers Públicas** (conforme escolha do projeto)
   - ✅ Simplicidade de configuração
   - ⚠️ Requer security lists rigorosas
   - ⚠️ Recomendado usar NSGs adicionais para controle granular

2. **Security Lists Recomendadas**
   - Restringir ingress apenas a tráfego necessário
   - Egress: Permitir apenas destinos essenciais (banco, Docker Hub, etc.)
   - Revisar regularmente as regras

3. **Certificados TLS**
   - Usar Cert-Manager com Let's Encrypt production
   - Configurar renovação automática
   - Monitorar expiração de certificados

4. **Network Policies**
   - Implementar Network Policies no Kubernetes
   - Restringir tráfego pod-a-pod
   - Bloquear tráfego inter-namespace não necessário

## Troubleshooting Geral

### Cluster não consegue acessar banco de dados

```bash
# 1. Verificar conectividade da subnet
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
nc -zv 164.152.38.179 1521

# 2. Verificar routes
# OCI Console → VCN → Route Tables → Verificar rota para 164.152.38.0/24

# 3. Verificar Security Lists
# OCI Console → VCN → Security Lists → Verificar regras de egress

# 4. Verificar LPG status
# OCI Console → VCN → Local Peering Gateways → Status deve ser "Peered"
```

### Pods não conseguem fazer resolve de DNS

```bash
# Verificar CoreDNS
kubectl get pods -n kube-system | grep coredns

# Verificar logs
kubectl logs -n kube-system deployment/coredns

# Testar DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

### LoadBalancer fica em "pending"

```bash
# Verificar eventos
kubectl get svc -n ingress-nginx
kubectl describe svc ingress-nginx-controller -n ingress-nginx

# Verificar if Cloud Controller Manager
kubectl get pods -n kube-system | grep cloud-controller
```

## Referências Externas

### Documentação Oracle
- [OKE Custom Cluster Creation](https://docs.oracle.com/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_Console_to_create_a_Custom_Cluster_with_Explicitly_Defined_Settings.htm)
- [Local Peering Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/localVCNpeering.htm)
- [VCN Best Practices](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/bestpracticesnetwork.htm)

### Documentação Kubernetes
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Cert-Manager](https://cert-manager.io/)
- [Prometheus Operator](https://prometheus-operator.dev/)

### Helm Charts
- [ingress-nginx/ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)
- [jetstack/cert-manager](https://artifacthub.io/packages/helm/cert-manager/cert-manager)
- [prometheus-community/kube-prometheus-stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)

## Suporte e Dúvidas

Consulte a seção "Troubleshooting" em cada documento específico para problemas em funcionalidades particulares.

Para questões gerais sobre OCI, consulte:
- [OCI Support Portal](https://support.oracle.com/)
- [Oracle Community](https://community.oracle.com/)

---

**Última atualização:** Fevereiro 2026
**Status:** Documentação completa
**Versão:** 1.0
