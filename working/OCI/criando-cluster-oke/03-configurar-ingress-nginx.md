# Configurar NGINX Ingress Controller

## Introdução

O **NGINX Ingress Controller** fornece roteamento profissional de tráfego HTTP/HTTPS em Kubernetes. Ele gerencia acesso externo aos serviços internos do cluster.

### Por Que Usar Ingress?

- ✅ Roteamento HTTP/HTTPS baseado em hostname e paths
- ✅ Load balancing automático entre replicas
- ✅ TLS/SSL nativo com suporte a múltiplos certificados
- ✅ Controle de acesso e rate limiting
- ✅ Melhor do que expor serviços NodePort ou LoadBalancer diretamente

### Arquitetura

```
Internet (público)
    ↓
 NGINX Ingress Controller (LoadBalancer Service)
    ↓
 Kubernetes Ingress Resources
    ↓
 Service → Pod (DeepTreasury App)
```

## Pré-Requisitos

- ✅ Cluster OKE criado (com VCN 10.2.0.0/16)
- ✅ kubectl configurado e conectado ao cluster
- ✅ Helm instalado (`helm version`)
- ✅ Subnet Load Balancer criada (10.2.20.0/24)
- ✅ Permissões para criar LoadBalancer services no OCI

## Parte 1: Instalar NGINX Ingress Controller

### Passo 1.1: Adicionar Repositório Helm

```bash
# Adicionar repositório NGINX oficial
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Atualizar cache de repositórios
helm repo update

# Verificar que o repositório foi adicionado
helm repo list | grep ingress-nginx
# Resultado esperado: ingress-nginx  https://kubernetes.github.io/ingress-nginx
```

### Passo 1.2: Criar Namespace

```bash
# Criar namespace para o ingress controller
kubectl create namespace ingress-nginx

# Verificar criação
kubectl get namespace ingress-nginx
```

### Passo 1.3: Instalar Chart Helm

```bash
# Instalar NGINX Ingress com configurações para OCI
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.service.type=LoadBalancer \
  --set controller.service.externalTrafficPolicy=Local \
  --set controller.metrics.enabled=true \
  --set controller.podAnnotations."prometheus\.io/scrape"=true \
  --set controller.podAnnotations."prometheus\.io/port"=10254 \
  --set-string controller.nodeSelector.workload-type=treasury

# Alternativas de instalação:
# - Com subnet específica para LB (se necessário)
# - Com valores customizados (nginx-values.yaml)
```

> ⏱️ Aguarde 1-2 minutos para o LoadBalancer receber IP externo.

### Passo 1.4: Verificar Instalação

```bash
# Listar pods do ingress-nginx
kubectl get pods -n ingress-nginx

# Resultado esperado:
# NAME                                        READY   STATUS
# ingress-nginx-controller-xxxx               1/1     Running
# ingress-nginx-admission-xxxx                1/1     Running

# Listar services
kubectl get svc -n ingress-nginx

# Resultado esperado - anote o EXTERNAL-IP:
# NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)
# ingress-nginx-controller   LoadBalancer   10.96.x.x    <IP-PUBLICO>  80:xxx, 443:xxx
```

## Parte 2: Configurar LoadBalancer Service no OCI

O NGINX Ingress Controller cria um LoadBalancer service que, por sua vez, cria um Load Balancer do OCI automaticamente.

### Passo 2.1: Verificar Load Balancer Criado

```bash
# Obter informações do service LoadBalancer
kubectl get svc -n ingress-nginx ingress-nginx-controller -o wide

# Armazenar o IP externo
export INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo "NGINX Ingress IP: $INGRESS_IP"
```

### Passo 2.2: Validar Load Balancer no OCI Console

1. Navegue até **Networking → Load Balancers**
2. Você deve ver um novo Load Balancer criado automaticamente
3. Verifique:
   - Status: "Active"
   - IP Público: Deve corresponder ao `EXTERNAL-IP` do kubectl
   - Subnet: `oke-lb-subnet` (10.2.20.0/24)
   - Listeners: Porta 80 (HTTP) e 443 (HTTPS)

> 💡 O OCI cria automaticamente um Load Balancer quando você cria um Kubernetes Service do tipo LoadBalancer.

## Parte 3: Configurar Ingress para DeepTreasury

### Passo 3.1: Criar Namespace da Aplicação

```bash
# Criar namespace para DeepTreasury
kubectl create namespace ts-dev

# Verificar
kubectl get namespace ts-dev
```

### Passo 3.2: Criar Ingress Resource

Crie um arquivo `ingress-treasure.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: deeptreasury-ingress
  namespace: ts-dev
  annotations:
    kubernetes.io/ingress.class: nginx
    # Anotações NGINX
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    # Rate limiting (opcional)
    nginx.ingress.kubernetes.io/limit-rps: "100"
    # Health check
    nginx.ingress.kubernetes.io/health-check-path: "/health"
spec:
  # Configuração de HTTP/HTTPS será adicionada na Parte 4
  rules:
    - host: "api.deeptreasury.com"  # SUBSTITUA COM SEU DOMÍNIO
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: deeptreasury-service
                port:
                  number: 80
```

Aplicar:

```bash
kubectl apply -f ingress-treasure.yaml

# Verificar
kubectl get ingress -n ts-dev
```

### Passo 3.3: Criar Service para a Aplicação

Se você ainda não tem um service, crie um arquivo `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deeptreasury-service
  namespace: ts-dev
spec:
  type: ClusterIP  # Não é necessário LoadBalancer, o Ingress cuida disso
  selector:
    app: deeptreasury
  ports:
    - protocol: TCP
      port: 80  # Porta do serviço interno
      targetPort: 8080  # Porta da aplicação no container
```

Aplicar:

```bash
kubectl apply -f service.yaml
```

## Parte 4: Configurar TLS/SSL com Cert-Manager

Para usar HTTPS (recomendado), você precisa de certificados. Consulte o documento **[`04-configurar-cert-manager.md`](./04-configurar-cert-manager.md)** para configuração completa.

Resumo para adicionar TLS ao Ingress:

```yaml
spec:
  tls:
    - hosts:
        - api.deeptreasury.com
      secretName: deeptreasury-tls  # Cert-Manager criará automaticamente
  rules:
    - host: api.deeptreasury.com
      # ... resto das rules
```

## Parte 5: Atualizar DNS

### Passo 5.1: Obter IP do Load Balancer

```bash
# Obter IP externo
kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Exemplo: 132.145.123.45
```

### Passo 5.2: Criar Registro DNS

Para seu domínio (ex: api.deeptreasury.com):

1. Acesse seu provedor de DNS (AWS Route53, Azure DNS, OCI DNS, etc.)
2. Crie um registro A:
   - **Host:** api.deeptreasury.com
   - **Type:** A
   - **Value:** 132.145.123.45 (IP obtido acima)
   - **TTL:** 300 (5 minutos para testes, 3600 para produção)

3. Aguarde a propagação de DNS (pode levar até 24 horas, geralmente minutos)

### Passo 5.3: Validar DNS

```bash
# Testar resolução
nslookup api.deeptreasury.com

# Ou com dig
dig api.deeptreasury.com

# Resultado esperado:
# api.deeptreasury.com. 300 IN A 132.145.123.45
```

## Parte 6: Testes e Validação

### Passo 6.1: Teste HTTP Básico

```bash
# Teste direto ao IP do Load Balancer
curl -H "Host: api.deeptreasury.com" http://132.145.123.45

# Ou via DNS (após DNS estar resolvido)
curl http://api.deeptreasury.com

# Resultado esperado: resposta da aplicação DeepTreasury
```

### Passo 6.2: Teste de Paths Múltiplos

Se sua Ingress tem múltiplos paths:

```bash
# Path 1
curl http://api.deeptreasury.com/api/users

# Path 2
curl http://api.deeptreasury.com/api/accounts
```

### Passo 6.3: Verificar Logs do NGINX

```bash
# Obter nome do pod NGINX
kubectl get pods -n ingress-nginx

# Ver logs
kubectl logs -n ingress-nginx ingress-nginx-controller-xxx

# Ou em tempo real
kubectl logs -n ingress-nginx ingress-nginx-controller-xxx -f
```

### Passo 6.4: Teste de Health Check

```bash
# Se sua aplicação tem endpoint /health
curl http://api.deeptreasury.com/health

# Resultado esperado: 200 OK
```

## Validação Completa

### Checklist NGINX Ingress

- [ ] Pod `ingress-nginx-controller` está em status "Running"
- [ ] Service `ingress-nginx-controller` tem IP externo (EXTERNAL-IP não é `<pending>`)
- [ ] Load Balancer no OCI Console está "Active"
- [ ] Service DeepTreasury está criado e endpoints estão saudáveis
- [ ] Ingress Resource está criado (`kubectl get ingress -n ts-dev`)
- [ ] DNS resolve para o IP do Load Balancer
- [ ] Curl para o domínio retorna resposta da aplicação

## Customizações Avançadas

### Adicionar Anotações NGINX

```yaml
metadata:
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "10"

    # Timeout
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"

    # Gzip compression
    nginx.ingress.kubernetes.io/enable-compression: "true"
    nginx.ingress.kubernetes.io/compression-level: "5"

    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"
```

### Múltiplos Domínios

```yaml
rules:
  - host: api.deeptreasury.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: deeptreasury-service
              port:
                number: 80

  - host: staging-api.deeptreasury.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: deeptreasury-staging-service
              port:
                number: 80
```

## Troubleshooting

### Problema: LoadBalancer IP fica em `<pending>`

**Causa:** Cloud Controller Manager não iniciou ou está com erro.

**Solução:**
```bash
# Verificar CCM
kubectl get pods -n kube-system | grep cloud-controller

# Se não existe ou está falhando:
# 1. Verifique eventos do cluster
kubectl describe nodes

# 2. Verifique logs
kubectl logs -n kube-system -l k8s-app=cloud-controller-manager
```

### Problema: Ingress mostra `<none>` para ADDRESS

**Causa:** Ingress Controller não está operacional.

**Solução:**
```bash
# Verificar controller pods
kubectl get pods -n ingress-nginx

# Ver eventos do Ingress
kubectl describe ingress deeptreasury-ingress -n ts-dev

# Se necessário, reinstale:
helm delete ingress-nginx -n ingress-nginx
# ... instale novamente (Passo 1.3)
```

### Problema: Tráfego não chega à aplicação

**Debug:**
```bash
# 1. Verificar se aplicação está rodando
kubectl get pods -n ts-dev
kubectl logs -n ts-dev <pod-name>

# 2. Verificar service endpoints
kubectl get endpoints -n ts-dev deeptreasury-service

# 3. Testar conectividade de pod para pod
kubectl run -it --rm debug --image=busybox -n ts-dev -- sh
wget -qO- deeptreasury-service  # Teste interno
```

### Problema: Certificado SSL não funciona

Veja: **[`04-configurar-cert-manager.md`](./04-configurar-cert-manager.md)**

## Próximos Passos

✅ NGINX Ingress Controller está instalado e funcionando

**Próximos passos:**
1. **[`04-configurar-cert-manager.md`](./04-configurar-cert-manager.md)** - Adicionar HTTPS/TLS
2. **[`05-configurar-monitoring.md`](./05-configurar-monitoring.md)** - Monitorar o cluster
3. **[`06-migrar-aplicacao.md`](./06-migrar-aplicacao.md)** - Migrar DeepTreasury

## Referências

- [NGINX Ingress Controller Official](https://kubernetes.github.io/ingress-nginx/)
- [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [Kubernetes Ingress API](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Helm Chart ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
