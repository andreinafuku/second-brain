# Diagrama da Arquitetura de Rede

## Visão Geral Completa

```
╔════════════════════════════════════════════════════════════════════════════╗
║                                                                            ║
║                          INTERNET PÚBLICO (0.0.0.0/0)                      ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
                                     ↓
                           DNS Resolution
                    (api.deeptreasury.com → 132.145.123.45)
                                     ↓
╔════════════════════════════════════════════════════════════════════════════╗
║                                                                            ║
║                        ORACLE CLOUD INFRASTRUCTURE                         ║
║                     Region: sa-saopaulo-1 (São Paulo)                     ║
║                                                                            ║
║  ┌──────────────────────────────────────────────────────────────────────┐ ║
║  │                                                                      │ ║
║  │                    VCN OKE (10.2.0.0/16) - NOVA                     │ ║
║  │                                                                      │ ║
║  │  ┌─ Internet Gateway ─────────────────────────────────────────────┐ │ ║
║  │  │ (Route: 0.0.0.0/0 → Internet)                                 │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Subnet: API Endpoint (10.2.0.0/28) - PUBLIC ─────────────────┐ │ ║
║  │  │ • 14 IPs disponíveis                                          │ │ ║
║  │  │ • Kubernetes API control plane (porta 6443)                   │ │ ║
║  │  │ • Security: Ingress TCP 6443 (0.0.0.0/0)                      │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Subnet: Workers (10.2.10.0/24) - PUBLIC ──────────────────────┐ │ ║
║  │  │ • 254 IPs disponíveis                                         │ │ ║
║  │  │ • 3 Worker Nodes com IPs públicos                             │ │ ║
║  │  │ • Pods DeepTreasury                                           │ │ ║
║  │  │ • CIDR interno: 10.244.0.0/16 (pods)                          │ │ ║
║  │  │                                                               │ │ ║
║  │  │  ┌─── Worker Node 1 ───────────────────────────────────────┐ │ │ ║
║  │  │  │ • IP: 10.2.10.x (privado)                               │ │ │ ║
║  │  │  │ • IP: 132.145.y.y (público)                             │ │ │ ║
║  │  │  │ • Pods:                                                  │ │ │ ║
║  │  │  │   - deeptreasury-xxx (10.244.0.x)                        │ │ │ ║
║  │  │  │   - coredns-xxx (10.244.0.y)                             │ │ │ ║
║  │  │  │   - kube-proxy-xxx                                       │ │ │ ║
║  │  │  └───────────────────────────────────────────────────────────┘ │ │ ║
║  │  │                                                               │ │ ║
║  │  │  ┌─── Worker Node 2 ───────────────────────────────────────┐ │ │ ║
║  │  │  │ • IP: 10.2.10.y (privado)                               │ │ │ ║
║  │  │  │ • IP: 132.145.z.z (público)                             │ │ │ ║
║  │  │  │ • Pods: idem...                                          │ │ │ ║
║  │  │  └───────────────────────────────────────────────────────────┘ │ │ ║
║  │  │                                                               │ │ ║
║  │  │  ┌─── Worker Node 3 ───────────────────────────────────────┐ │ │ ║
║  │  │  │ • IP: 10.2.10.z (privado)                               │ │ │ ║
║  │  │  │ • IP: 132.145.w.w (público)                             │ │ │ ║
║  │  │  │ • Pods: idem...                                          │ │ │ ║
║  │  │  └───────────────────────────────────────────────────────────┘ │ │ ║
║  │  │                                                               │ │ ║
║  │  │ • Security List: Ingress TCP 1521 to 164.152.38.0/24        │ │ ║
║  │  │ • Route: 164.152.38.0/24 → LPG (para banco de dados)        │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Subnet: Load Balancer (10.2.20.0/24) - PUBLIC ──────────────┐ │ ║
║  │  │ • 254 IPs disponíveis                                        │ │ ║
║  │  │ • NGINX Ingress Controller (LoadBalancer Service)            │ │ ║
║  │  │   - IP interno: 10.2.20.x                                    │ │ ║
║  │  │   - IP público (via Load Balancer OCI): 132.145.123.45      │ │ ║
║  │  │   - Porta 80: HTTP (redireciona para HTTPS)                  │ │ ║
║  │  │   - Porta 443: HTTPS (TLS, certificado Let's Encrypt)       │ │ ║
║  │  │                                                              │ │ ║
║  │  │ • Security: Ingress TCP 80, 443 (0.0.0.0/0)                 │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Local Peering Gateway (LPG-OKE) ────────────────────────────┐ │ ║
║  │  │ • Conecta VCN OKE ao VCN xrt-interno                         │ │ ║
║  │  │ • Status: PEERED                                             │ │ ║
║  │  │ • Route table: 164.152.38.0/24 → LPG                         │ │ ║
║  │  │ • Tráfego: Workers → LPG → xrt-interno → Database           │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  └──────────────────────────────────────────────────────────────────────┘ │ ║
║                                                                            ║
║                             LOCAL PEERING LINK                            ║
║                    (Low latency, 0 cost data transfer)                     ║
║                                                                            ║
║  ┌──────────────────────────────────────────────────────────────────────┐ ║
║  │                                                                      │ ║
║  │               VCN xrt-interno (10.0.0.0/16) - EXISTENTE            │ ║
║  │                                                                      │ ║
║  │  ┌─ Local Peering Gateway (LPG-xrt) ────────────────────────────┐ │ ║
║  │  │ • Recebe conexões de LPG-OKE                               │ │ ║
║  │  │ • Status: PEERED                                           │ │ ║
║  │  │ • Route table: 10.2.0.0/16 → LPG                           │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Subnet: Database (164.152.38.0/24) - PRIVATE ────────────────┐ │ ║
║  │  │ • Oracle Database Server                                    │ │ ║
║  │  │   - IP: 164.152.38.179                                      │ │ ║
║  │  │   - Porta: 1521 (Oracle Listener)                           │ │ ║
║  │  │   - Status: ATIVO, recebendo conexões de OKE                │ │ ║
║  │  │                                                             │ │ ║
║  │  │ • Security List: Ingress TCP 1521 de 10.2.10.0/24           │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  │  ┌─ Outras Subnets (não mapeadas aqui) ──────────────────────────┐ │ ║
║  │  │ • NAT Gateway, VPN Gateway, etc.                            │ │ ║
║  │  └────────────────────────────────────────────────────────────────┘ │ ║
║  │                                                                      │ ║
║  └──────────────────────────────────────────────────────────────────────┘ │ ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

## Diagrama de Fluxo de Tráfego

```
Cliente (Internet)
   ↓
   ↓ Request HTTPS (api.deeptreasury.com:443)
   ↓
OCI Load Balancer
   ↓ TCP 443 → 10.2.20.0/24
   ↓
NGINX Ingress Controller
   ↓ route: /api/users → service:deeptreasury-service:80
   ↓
Kubernetes Service (Endpoint selection via kube-proxy)
   ↓
Pod DeepTreasury (10.244.0.x) NO NODE 1, 2 ou 3
   ↓
   ├─ Request: GET /api/users
   │  ↓
   │  Resposta: JSON (usuários)
   │  ↓
   │  Response: HTTP 200 ← volta para cliente
   │
   └─ Request: SELECT * FROM usuarios WHERE company_id=X
      ↓
      TCP 1521 → 164.152.38.179 (Database)
      ↓
      Route via LPG: 10.2.10.0/24 → LPG-OKE → LPG-xrt → 164.152.38.0/24
      ↓
      Oracle Database recebe query
      ↓
      Response: Dataset ← volta pela LPG
      ↓
      Pod recebe dados ← formata JSON ← envia ao cliente
```

## Tabela de Endereçamento IP

### VCN OKE (10.2.0.0/16)

| Recurso | IP / CIDR | Descrição | Hosts/Máquinas |
|---------|-----------|-----------|-----------------|
| **VCN** | 10.2.0.0/16 | Rede principal | - |
| **Subnet API** | 10.2.0.0/28 | Kubernetes API | 14 disponíveis |
| **Subnet Workers** | 10.2.10.0/24 | Worker nodes | 3 nodes + 251 outros |
| **Subnet LB** | 10.2.20.0/24 | NGINX LoadBalancer | 1 LB + 253 outros |
| **Pod CIDR** | 10.244.0.0/16 | Pods (interno K8s) | até 65,536 pods |
| **Service CIDR** | 10.96.0.0/12 | Services (interno K8s) | até 1M services |

### VCN xrt-interno (10.0.0.0/16)

| Recurso | IP / CIDR | Descrição | Hosts/Máquinas |
|---------|-----------|-----------|-----------------|
| **VCN** | 10.0.0.0/16 | Rede existente | - |
| **Subnet Database** | 164.152.38.0/24 | Database subnet | até 254 |
| **Database** | 164.152.38.179 | Oracle Database | 1 host |
| **Database Port** | :1521 | Oracle Listener | TCP |

### Sumarização

| Componente | IPv4 Privado | IPv4 Público | Protocolo | Porta |
|-----------|------------|-----------|-----------|-------|
| **Kubernetes API** | 10.2.0.x | - | TCP | 6443 |
| **Worker Node 1** | 10.2.10.x | 132.145.y.y | TCP/ICMP | 22 (SSH) |
| **Worker Node 2** | 10.2.10.y | 132.145.z.z | TCP/ICMP | 22 (SSH) |
| **Worker Node 3** | 10.2.10.z | 132.145.w.w | TCP/ICMP | 22 (SSH) |
| **NGINX LoadBalancer** | 10.2.20.x | 132.145.123.45 | TCP | 80, 443 |
| **Pod DeepTreasury** | 10.244.0.x | - | TCP | 8080, 9090 |
| **Oracle Database** | 164.152.38.179 | - | TCP | 1521 |

## Diagrama de Componentes Kubernetes

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLUSTER KUBERNETES                          │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐  │
│ │                   kube-system namespace                   │  │
│ │                                                           │  │
│ │  • coredns-xxx (DNS interno)                             │  │
│ │  • kube-proxy-xxx (network proxy, um por node)            │  │
│ │  • etcd-xxx (database do cluster)                        │  │
│ │  • kube-apiserver (API do cluster)                       │  │
│ │  • kube-controller-manager (controllers)                 │  │
│ │  • kube-scheduler (schedule de pods)                     │  │
│ │  • cloud-controller-manager (integração OCI)            │  │
│ │                                                           │  │
│ └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐  │
│ │              ingress-nginx namespace                      │  │
│ │                                                           │  │
│ │  Service: LoadBalancer (132.145.123.45)                  │  │
│ │    │                                                      │  │
│ │    └─ ingress-nginx-controller (NGINX pod)               │  │
│ │       • Listen: 0.0.0.0:80, 0.0.0.0:443                 │  │
│ │       • Rule: api.deeptreasury.com → service             │  │
│ │       • TLS: Let's Encrypt cert                          │  │
│ │                                                           │  │
│ └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐  │
│ │               cert-manager namespace                      │  │
│ │                                                           │  │
│ │  • cert-manager (gerencia certificados)                  │  │
│ │  • cert-manager-webhook (validação)                      │  │
│ │  • cert-manager-cainjector (CA injection)                │  │
│ │                                                           │  │
│ │  ClusterIssuer: letsencrypt-prod                         │  │
│ │    └─ Certificate: deeptreasury-tls (Secret)             │  │
│ │                                                           │  │
│ └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐  │
│ │             monitoring namespace                          │  │
│ │                                                           │  │
│ │  • prometheus-0 (scrape de métricas)                     │  │
│ │    └─ Scrape targets: kubelet, node-exporter, pods       │  │
│ │  • grafana (visualização)                                │  │
│ │  • alertmanager (alertas)                                │  │
│ │  • node-exporter-xxx (um por node)                       │  │
│ │  • kube-state-metrics (métricas K8s)                     │  │
│ │                                                           │  │
│ └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐  │
│ │                 ts-dev namespace (APP)                    │  │
│ │                                                           │  │
│ │  Deployment: deeptreasury                                │  │
│ │    ├─ Pod: deeptreasury-abc-123 (node-1)                 │  │
│ │    │  • Container: deeptreasury (8080, 9090)             │  │
│ │    │  • Env: DB_HOST=164.152.38.179, DB_PORT=1521        │  │
│ │    │  • Volume: config, secrets                          │  │
│ │    │                                                      │  │
│ │    ├─ Pod: deeptreasury-def-456 (node-2)                 │  │
│ │    │  • idem...                                           │  │
│ │    │                                                      │  │
│ │    └─ Pod: deeptreasury-ghi-789 (node-3)                 │  │
│ │       • idem...                                           │  │
│ │                                                           │  │
│ │  Service: deeptreasury-service                           │  │
│ │    ├─ Port http: 80 → 8080                               │  │
│ │    └─ Port metrics: 9090 → 9090                          │  │
│ │                                                           │  │
│ │  Ingress: deeptreasury-ingress                           │  │
│ │    ├─ Host: api.deeptreasury.com                         │  │
│ │    ├─ Path: / → service:deeptreasury-service:80          │  │
│ │    ├─ TLS Secret: deeptreasury-tls (auto gerado)         │  │
│ │    └─ Cert Manager: Yes (letsencrypt-prod)               │  │
│ │                                                           │  │
│ │  ConfigMap: app-config                                   │  │
│ │    ├─ LOG_LEVEL: INFO                                    │  │
│ │    └─ Outros: conforme necessário                        │  │
│ │                                                           │  │
│ │  Secret: db-credentials                                  │  │
│ │    ├─ username: <hidden>                                 │  │
│ │    └─ password: <hidden>                                 │  │
│ │                                                           │  │
│ │  Secret: ocir-secret (docker registry)                   │  │
│ │  Secret: deeptreasury-tls (TLS certificate)              │  │
│ │                                                           │  │
│ └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Fluxo de Resolução de DNS

```
Cliente executa: curl https://api.deeptreasury.com

1. Resolver de DNS local (ou ISP)
   └─ Query: A record para api.deeptreasury.com
      └─ Resposta: 132.145.123.45 (IP público do Load Balancer)

2. Cliente faz conexão TCP
   └─ Conecta em 132.145.123.45:443
      └─ Chega ao OCI Load Balancer

3. Load Balancer encaminha para backend
   └─ Backend: NGINX Ingress Controller (10.2.20.x)
      └─ Porta interna: 443

4. NGINX recebe request
   └─ Analisa Host header: api.deeptreasury.com
   └─ Encontra Ingress rule: api.deeptreasury.com → deeptreasury-service
      └─ Faz request ao Service (10.96.x.x:80)

5. Kube-proxy intercept no pod local
   └─ Realiza NAT: 10.96.x.x:80 → 10.244.0.x:8080
   └─ Encaminha para Pod DeepTreasury

6. Pod responde
   └─ Response retorna: 10.244.0.x → NGINX → Load Balancer → Cliente
```

## Fluxo de Conectividade ao Banco de Dados

```
Pod DeepTreasury (10.244.0.x) executa:
new OracleConnection("Data Source=164.152.38.179:1521/ORCL")

1. Pod resolve hostname/IP
   └─ 164.152.38.179 é resolvido (IP já conhecido)

2. Pod cria TCP connection
   └─ Source: 10.244.0.x (pod)
   └─ Destination: 164.152.38.179:1521 (database)

3. Kernel de rede processa
   └─ Packet: 10.244.0.x → 164.152.38.179
   └─ Source NAT: 10.244.0.x → 10.2.10.x (worker node IP)
      └─ Porque pod network usa overlay (CIDR 10.244.0.0/16)

4. Kernel do Worker Node encaminha
   └─ Consulta route table local
   └─ Procura match: 164.152.38.179 pertence a 164.152.38.0/24?
   └─ SIM! → Route: 164.152.38.0/24 → VCN Gateway (LPG)

5. LPG encaminha para VCN xrt-interno
   └─ Packet sai pela LPG do lado OKE
   └─ Entra pela LPG do lado xrt-interno
   └─ Route table xrt-interno: 10.2.10.x deve ir para aqui
      └─ Porque source IP é 10.2.10.x

6. Subnet database recebe packet
   └─ Security List: permite TCP 1521 de 10.2.10.0/24?
   └─ SIM! → Packet liberado

7. Oracle Database recebe e responde
   └─ Response: 164.152.38.179 → 10.2.10.x
   └─ Retorna pela LPG

8. Pod recebe response
   └─ Dest NAT: 10.2.10.x → 10.244.0.x (volta ao pod)
   └─ Connection estabelecida!
```

## Checklist de Segurança

### Security Lists Configuradas

- [x] **Subnet API (10.2.0.0/28)**
  - Ingress: TCP 6443 (Kubernetes API) de 0.0.0.0/0
  - Egress: All protocols para 0.0.0.0/0

- [x] **Subnet Workers (10.2.10.0/24)**
  - Ingress: TCP 22 (SSH) de seu_ip/32 ou 0.0.0.0/0
  - Ingress: TCP 443 (HTTPS) de 0.0.0.0/0 (tráfego de pod)
  - Egress: TCP 1521 para 164.152.38.0/24 (database)
  - Egress: UDP 53 (DNS) para 0.0.0.0/0
  - Egress: All protocols para 0.0.0.0/0 (internet)

- [x] **Subnet Load Balancer (10.2.20.0/24)**
  - Ingress: TCP 80 (HTTP) de 0.0.0.0/0
  - Ingress: TCP 443 (HTTPS) de 0.0.0.0/0
  - Egress: All protocols para 0.0.0.0/0

- [x] **Subnet Database (164.152.38.0/24) - xrt-interno**
  - Ingress: TCP 1521 (Oracle) de 10.2.10.0/24 (OKE workers)
  - Existir: Outras regras necessárias (admin access, etc.)

### Network Policies Kubernetes (Opcional)

```yaml
# Bloquear tráfego por default em ts-dev
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: ts-dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Permitir apenas Ingress do NGINX
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-ingress
  namespace: ts-dev
spec:
  podSelector:
    matchLabels:
      app: deeptreasury
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080

---
# Permitir Egress ao banco de dados
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-database
  namespace: ts-dev
spec:
  podSelector:
    matchLabels:
      app: deeptreasury
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 164.152.38.0/24
    ports:
    - protocol: TCP
      port: 1521
```

## Monitoramento de Tráfego

### Métricas Prometheus para Observar

```
# Tráfego HTTP
rate(http_request_duration_seconds_bucket[5m])

# Conexões ao banco
db_connection_pool_size
db_connection_active
db_query_duration_seconds

# Tráfego de rede
container_network_transmit_bytes_total
container_network_receive_bytes_total

# Pod network latency
rate(pod_network_latency_seconds[5m])
```

### Comandos para Debug de Rede

```bash
# Ver rotas no node
kubectl debug node/<node-name> -it --image=ubuntu
  └─ ip route show
  └─ iptables-save | grep 164.152
  └─ ss -tan | grep 1521

# Capturar tráfego
kubectl exec <pod-name> -it -- tcpdump -i any -n 'tcp port 1521'

# Teste de latência
kubectl exec <pod-name> -it -- ping 164.152.38.179

# Teste de conectividade
kubectl exec <pod-name> -it -- nc -zv 164.152.38.179 1521
```

## Referências Visuais

### Port Mapping

```
External           |  Kubernetes       |  Application
Client             |  Service/Pod      |  Container
───────────────────────────────────────────────────────
:80 (HTTP)    ─→   | NGINX :80    ─→   | DeepTreasury :8080
:443 (HTTPS)  ─→   | NGINX :443   ─→   | DeepTreasury :8080
              │    | NGINX         │
              └────│ (TLS termination) │
                   |                   |
              ─→   | Prometheus :9090 ◄─ DeepTreasury :9090
              (metrics scrape)        (metrics endpoint)
```

### Addressing Layers

```
Layer              Range / CIDR              Notes
────────────────────────────────────────────────────
VCN OKE            10.2.0.0/16              Virtual Cloud Network
 ├─ Subnet API     10.2.0.0/28              API control plane
 ├─ Subnet Workers 10.2.10.0/24             Compute nodes
 └─ Subnet LB      10.2.20.0/24             Load balancer

Pod CIDR           10.244.0.0/16            Overlay network (K8s)
Service CIDR       10.96.0.0/12             Virtual IPs (K8s)

VCN xrt-interno    10.0.0.0/16              Database VCN
 └─ Subnet DB      164.152.38.0/24          Oracle Database
    └─ DB Instance 164.152.38.179:1521      Oracle DB endpoint

Public IPs         132.145.x.x/32           Assigned by OCI
 └─ Load Balancer  132.145.123.45           NGINX controller
 └─ Worker Nodes   132.145.y.y, z.z, w.w   SSH access
```

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
