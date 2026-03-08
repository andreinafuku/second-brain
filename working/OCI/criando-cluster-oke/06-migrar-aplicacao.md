# Migrar Aplicação DeepTreasury para Novo Cluster OKE

## Introdução

Este guia fornece um passo a passo para migrar a aplicação DeepTreasury do cluster OKE antigo para o novo cluster com VCN customizada (10.2.0.0/16).

### Motivação da Migração

- ✅ Novo cluster tem VCN sem conflito de CIDR (10.2.0.0/16 vs 10.0.0.0/16 do antigo)
- ✅ Conectividade via Local Peering Gateway para banco de dados
- ✅ Infraestrutura moderna com NGINX Ingress + Cert-Manager
- ✅ Monitoring completo com Prometheus + Grafana

### Estratégia de Migração

```
Fase 1: Preparação (dia 0)
  ↓
Fase 2: Deploy no novo cluster (dia 1)
  ↓
Fase 3: Testes e validação (dia 1-2)
  ↓
Fase 4: Cutover (mudança para produção) (dia 2-3)
  ↓
Fase 5: Monitoramento (dia 3-14)
  ↓
Fase 6: Descomissionamento do cluster antigo (dia 14+)
```

## Fase 1: Preparação

### Passo 1.1: Documentar Configuração Atual

Extraia todas as configurações do cluster antigo:

```bash
# Exportar todos os namespaces
kubectl get namespaces -o yaml > namespaces-backup.yaml

# Exportar deployments
kubectl get deployments -A -o yaml > deployments-backup.yaml

# Exportar services
kubectl get services -A -o yaml > services-backup.yaml

# Exportar ingress
kubectl get ingress -A -o yaml > ingress-backup.yaml

# Exportar configmaps (NÃO contém secrets)
kubectl get configmaps -A -o yaml > configmaps-backup.yaml

# IMPORTANTE: Exportar secrets SEPARADAMENTE (não incluir em backup geral)
# kubectl get secrets -A -o yaml > secrets-backup.yaml
# ⚠️ Mantenha secrets-backup.yaml SEGURO (criptografar, acesso restrito)
```

### Passo 1.2: Identificar Dependências

Documente:

1. **Imagens Docker**
   ```bash
   kubectl get pods -A -o jsonpath='{..image}' | tr -s '[[:space:]]' '\n' | sort | uniq
   ```

2. **Volumes e Storage**
   ```bash
   kubectl get pvc -A
   kubectl get pv
   ```

3. **Environment Variables e Configs**
   ```bash
   kubectl get configmaps -A -o yaml
   ```

4. **Secrets (valores NOT inclusos)**
   ```bash
   # Liste nomes, NÃO valores
   kubectl get secrets -A
   ```

5. **Serviços externos**
   - Banco de dados: 164.152.38.179:1521
   - Cache/Redis (se usado)
   - APIs externas

### Passo 1.3: Criar Plano de Cutover

Documente o plano:

```
PLANO DE CUTOVER - DeepTreasury

Data Estimada: [DATA]
Janela de Manutenção: [HH:MM] - [HH:MM]

Sequência:
1. Parar ingress do cluster antigo (10:00)
2. Verificar que tráfego novo vai para novo cluster (10:05)
3. Manter cluster antigo em standby por 7 dias (10:10-[DATA+7])
4. Deletar cluster antigo após validação (se tudo OK)

Contacts:
- Líder técnico: [NOME]
- DBA: [NOME]
- On-call: [CONTATO]

Rollback plan:
- Se problema, reverter DNS em 2 minutos
- Cluster antigo manterá dados por 7 dias
```

## Fase 2: Deploy no Novo Cluster

### Passo 2.1: Conectar ao Novo Cluster

```bash
# Mudar contexto para novo cluster
kubectl config use-context <NOVO_CLUSTER_CONTEXT>

# Verificar
kubectl cluster-info
kubectl get nodes
```

### Passo 2.2: Criar Namespaces

```bash
# Criar namespace da aplicação
kubectl create namespace ts-dev

# Criar namespaces adicionais conforme necessário
kubectl create namespace monitoring  # Se não existe
kubectl create namespace ingress-nginx  # Se não existe
```

### Passo 2.3: Criar Secrets

⚠️ **IMPORTANTE:** Secrets devem ser criados manualmente ou via ferramentas seguras (Sealed Secrets, External Secrets, etc.)

```bash
# Database credentials
kubectl create secret generic db-credentials \
  --from-literal=username=<DB_USER> \
  --from-literal=password=<DB_PASSWORD> \
  -n ts-dev

# Docker registry (OCIR ou privado)
kubectl create secret docker-registry ocir-secret \
  --docker-server=<REGISTRY_URL> \
  --docker-username=<USERNAME> \
  --docker-password=<PASSWORD> \
  --docker-email=<EMAIL> \
  -n ts-dev

# Certificados, tokens, etc.
kubectl create secret generic app-tokens \
  --from-literal=api-key=<API_KEY> \
  --from-literal=jwt-secret=<JWT_SECRET> \
  -n ts-dev
```

### Passo 2.4: Criar ConfigMaps

```bash
# Aplicar configmaps do backup
# IMPORTANTE: Atualizar IPs/hosts para novo ambiente se necessário
kubectl apply -f configmaps-backup.yaml -n ts-dev

# Ou criar manualmente
kubectl create configmap app-config \
  --from-literal=DB_HOST=164.152.38.179 \
  --from-literal=DB_PORT=1521 \
  --from-literal=LOG_LEVEL=INFO \
  -n ts-dev
```

### Passo 2.5: Deploy da Aplicação

Crie/atualize `deployment-deeptreasury.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deeptreasury
  namespace: ts-dev
  labels:
    app: deeptreasury
    version: v1
spec:
  replicas: 3  # Múltiplas instâncias para HA
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: deeptreasury
  template:
    metadata:
      labels:
        app: deeptreasury
        workload-type: treasury
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      # Pull secrets para OCIR
      imagePullSecrets:
        - name: ocir-secret

      # Affinity para distribuir pods em diferentes nodes
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - deeptreasury
                topologyKey: kubernetes.io/hostname

      containers:
        - name: deeptreasury
          image: <REGISTRY>/deeptreasury:latest  # ALTERE COM SUAS IMAGENS
          imagePullPolicy: IfNotPresent

          # Portas
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: metrics
              containerPort: 9090
              protocol: TCP

          # Environment
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: "Production"
            - name: ASPNETCORE_URLS
              value: "http://+:8080"
            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: LOG_LEVEL

          # Credentials do banco
          envFrom:
            - secretRef:
                name: db-credentials
            - configMapRef:
                name: app-config

          # Health checks
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

          # Resources
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 1000m
              memory: 2Gi

          # Security
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
```

Aplicar:

```bash
kubectl apply -f deployment-deeptreasury.yaml

# Verificar deployment
kubectl get deployment -n ts-dev
kubectl get pods -n ts-dev

# Aguardar até todos pods ficarem "Running"
kubectl wait --for=condition=available --timeout=300s deployment/deeptreasury -n ts-dev
```

### Passo 2.6: Criar Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deeptreasury-service
  namespace: ts-dev
  labels:
    app: deeptreasury
spec:
  type: ClusterIP
  selector:
    app: deeptreasury
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
    - name: metrics
      port: 9090
      targetPort: metrics
      protocol: TCP
  sessionAffinity: ClientIP  # Sticky sessions se necessário
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

Aplicar:

```bash
kubectl apply -f service-deeptreasury.yaml

# Verificar service
kubectl get svc -n ts-dev
```

### Passo 2.7: Criar Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: deeptreasury-ingress
  namespace: ts-dev
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/limit-rps: "100"
spec:
  tls:
    - hosts:
        - api.deeptreasury.com
      secretName: deeptreasury-tls
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
```

Aplicar:

```bash
kubectl apply -f ingress-deeptreasury.yaml

# Verificar ingress
kubectl get ingress -n ts-dev
kubectl describe ingress deeptreasury-ingress -n ts-dev
```

## Fase 3: Testes e Validação

### Passo 3.1: Testar Conectividade ao Banco de Dados

```bash
# Port-forward ao pod da aplicação
kubectl port-forward -n ts-dev deployment/deeptreasury 8080:8080 &

# Testar endpoint de health
curl -v http://localhost:8080/health

# Resultado esperado: 200 OK
```

### Passo 3.2: Testar Endpoints da Aplicação

```bash
# Via port-forward
curl http://localhost:8080/api/users
curl http://localhost:8080/api/accounts

# Via NGINX (se DNS já está apontado)
# curl https://api.deeptreasury.com/api/users
```

### Passo 3.3: Verificar Logs

```bash
# Logs da aplicação
kubectl logs -n ts-dev deployment/deeptreasury -f

# Logs do NGINX Ingress
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller -f

# Ver eventos
kubectl get events -n ts-dev --sort-by='.lastTimestamp'
```

### Passo 3.4: Verificar Métricas (Prometheus)

```bash
# Port-forward ao Prometheus
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090 &

# Acesse http://localhost:9090 e procure por métricas DeepTreasury
# Exemplo: up{job="deeptreasury"}
```

### Passo 3.5: Validação de Performance

```bash
# Teste de carga (opcional, com ferramentas como Apache Bench)
ab -n 1000 -c 10 http://localhost:8080/

# Ou com curl loop
for i in {1..100}; do curl -s http://localhost:8080/health; done
```

## Fase 4: Cutover (Mudança para Produção)

### Passo 4.1: Preparação Final

```bash
# Dia anterior ao cutover:
# 1. Executar backup do banco de dados
# 2. Notificar usuários sobre manutenção
# 3. Sincronizar dados se necessário
# 4. Testar rollback procedure
```

### Passo 4.2: Parar Tráfego no Cluster Antigo

```bash
# Switch contexto para cluster antigo
kubectl config use-context <ANTIGO_CLUSTER_CONTEXT>

# Remover/desabilitar Ingress antigo
kubectl delete ingress deeptreasury-ingress -n ts-dev
# Ou editar para desabilitar anotação

# Verificar que aplicação não recebe mais tráfego
kubectl logs -n ts-dev deployment/deeptreasury | tail -20
```

### Passo 4.3: Atualizar DNS

Atualize seu DNS para apontar para novo Load Balancer:

```bash
# Obter novo IP do Load Balancer
kubectl config use-context <NOVO_CLUSTER_CONTEXT>

kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Exemplo: 132.145.123.45

# Atualizar DNS (seu provedor):
# api.deeptreasury.com A 132.145.123.45
# TTL: 300 segundos
```

### Passo 4.4: Monitorar Transição

```bash
# Monitor novo cluster
kubectl config use-context <NOVO_CLUSTER_CONTEXT>

# Verificar pods
watch -n 5 'kubectl get pods -n ts-dev'

# Verificar logs
kubectl logs -n ts-dev deployment/deeptreasury -f

# Verificar NGINX
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller -f
```

### Passo 4.5: Validação Pós-Cutover

```bash
# 1. Verificar resolução de DNS
nslookup api.deeptreasury.com

# 2. Teste de conectividade
curl -v https://api.deeptreasury.com/health

# 3. Verificar certificado TLS
openssl s_client -connect api.deeptreasury.com:443 -showcerts </dev/null

# 4. Teste de negócio (funcionalidade real)
# - Login como usuário
# - Executar operações críticas
# - Verificar dados no banco

# 5. Monitorar métricas
# - CPU/Memory usage normal?
# - Request latency aceitável?
# - Error rate próximo de zero?
```

## Fase 5: Monitoramento Pós-Cutover (7-14 dias)

### Passo 5.1: Daily Health Checks

```bash
# Checklist diário:

# 1. Pods running?
kubectl get pods -n ts-dev
kubectl describe pod <POD_NAME> -n ts-dev

# 2. Eventos de erro?
kubectl get events -n ts-dev | grep -i error

# 3. CPU/Memory usage
kubectl top pod -n ts-dev
kubectl top nodes

# 4. Certificado TLS válido?
echo | openssl s_client -connect api.deeptreasury.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# 5. Métricas Prometheus
# Prometheus HTTP error rate < 1%
# Database connection pool healthy
# Request latency p99 < 1s

# 6. Logs de erro?
kubectl logs -n ts-dev deployment/deeptreasury | grep -i error | head -20
```

### Passo 5.2: Backup do Novo Cluster

```bash
# Manter backups regulares
kubectl get all -n ts-dev -o yaml > ts-dev-day-5-backup.yaml

# Ou usar ferramentas de backup (Velero, etc.)
```

### Passo 5.3: Comunicação com Stakeholders

- Enviar daily reports
- Reportar qualquer anomalia
- Manter cluster antigo em standby

## Fase 6: Descomissionamento do Cluster Antigo

Apenas após 7-14 dias sem problemas:

### Passo 6.1: Backup Final

```bash
# Switch ao cluster antigo
kubectl config use-context <ANTIGO_CLUSTER_CONTEXT>

# Backup completo
kubectl get all -A -o yaml > antigo-cluster-final-backup.yaml

# Backup de dados (banco de dados)
# Executar backup full do banco Oracle
```

### Passo 6.2: Documentar Configurações

```bash
# Exportar todas as configurações para histórico
kubectl get all -A -o yaml > antigo-cluster-config-archive.yaml
```

### Passo 6.3: Deletar Cluster Antigo

```bash
# OCI Console → Kubernetes Clusters
# Selecione cluster antigo
# Clique em Delete
# Confirme

# Isso vai deletar:
# - Cluster
# - Node pools
# - LoadBalancer
# - Mas NÃO a VCN (poderia ser usada para outro cluster)
```

### Passo 6.4: Limpeza de Recursos OCI

```bash
# Opcionalmente, deletar VCN antiga (se não usada)
# OCI Console → Virtual Cloud Networks
# Selecione VCN antigua
# Delete (cascata deletará subnets e security lists também)

# Remover snapshots, backups não mais necessários
```

## Rollback (Se Necessário)

Se algo der errado durante/após cutover:

### Opção 1: Quick Rollback (Primeiras horas)

```bash
# 1. DNS switch de volta ao antigo cluster
# (seu provedor de DNS, TTL 5 minutos)

# 2. Cluster antigo continua rodando com aplicação anterior
# Não há downtime para usuários

# 3. Investigar problema no novo cluster
# Manter novo cluster rodando para debug

# Tempo total: ~5 minutos (tempo de propagação DNS)
```

### Opção 2: Data/Sync Rollback (Se dados foram modificados)

```bash
# 1. Parar novo cluster
# 2. Restaurar banco de dados do backup pré-migration
# 3. Voltar ao cluster antigo
# 4. Executar sync de dados se necessário

# Tempo: 30-60 minutos (dependendo do tamanho do banco)
```

## Validação Completa

### Checklist Final de Migração

- [ ] Novo cluster OKE criado
- [ ] VCN customizada (10.2.0.0/16) sem conflito de CIDR
- [ ] Local Peering Gateway configurado
- [ ] Pods DeepTreasury rodando em número correto
- [ ] Service e Ingress criados
- [ ] NGINX Ingress Controller ativo
- [ ] Cert-Manager emitiu certificado TLS válido
- [ ] DNS aponta para novo Load Balancer
- [ ] Testes de conectividade ao banco bem-sucedidos
- [ ] Endpoints da API retornam dados corretos
- [ ] Health checks passando
- [ ] Métricas Prometheus sendo coletadas
- [ ] Alertas configurados e funcionando
- [ ] Logs não mostram erros críticos
- [ ] Performance comparável ou melhorada
- [ ] Cluster antigo em standby por 7+ dias
- [ ] Cluster antigo deletado após validação

## Próximos Passos

✅ Migração concluída com sucesso!

**Após migração:**
1. Continuar monitorando por 30 dias
2. Otimizar dashboards e alertas
3. Documentar lições aprendidas
4. Planejar melhorias futuras

## Referências

- [Kubernetes Best Practices - Migration](https://kubernetes.io/docs/concepts/configuration/overview/)
- [OCI OKE Migration Guide](https://docs.oracle.com/iaas/Content/ContEng/Tasks/contengmigration.htm)
- [Blue-Green Deployments in Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
