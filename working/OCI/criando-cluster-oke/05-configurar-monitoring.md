# Configurar Monitoring com Prometheus e Grafana

## Introdução

O **kube-prometheus-stack** fornece um stack completo de monitoramento:
- **Prometheus:** Coleta de métricas
- **Grafana:** Visualização e dashboards
- **AlertManager:** Gerenciamento de alertas
- **Node-Exporter:** Métricas do sistema

### Por Que Usar Monitoring?

- ✅ Visibilidade completa do cluster e aplicação
- ✅ Alertas proativos de problemas
- ✅ Histórico de performance
- ✅ Dashboards customizados
- ✅ Debugging facilitado

### Arquitetura

```
Nodes/Pods (exportam métricas)
    ↓
 Prometheus (coleta a cada 30s)
    ↓
 Grafana (visualiza dados)
    ↓
 AlertManager (envia alertas)
```

## Pré-Requisitos

- ✅ Cluster OKE criado
- ✅ kubectl configurado
- ✅ Helm instalado
- ✅ Pelo menos 2GB de storage persistente disponível
- ✅ Acesso ao OCI Console para monitoramento de recursos

## Parte 1: Instalar kube-prometheus-stack

### Passo 1.1: Adicionar Repositório Helm

```bash
# Adicionar repositório prometheus-community
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Atualizar cache
helm repo update

# Verificar
helm repo list | grep prometheus-community
```

### Passo 1.2: Criar Namespace

```bash
# Namespace para monitoring
kubectl create namespace monitoring

# Verificar
kubectl get namespace monitoring
```

### Passo 1.3: Criar Values File (Configuração Customizada)

Crie um arquivo `kube-prometheus-values.yaml`:

```yaml
# Prometheus
prometheus:
  prometheusSpec:
    retention: 30d  # Manter dados por 30 dias
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi  # 10GB para dados Prometheus
    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: 1000m
        memory: 4Gi

# Grafana
grafana:
  enabled: true
  adminPassword: "ChangeMeToSecurePassword"  # ALTERE PARA SENHA SEGURA
  persistence:
    enabled: true
    storageClassName: standard
    size: 5Gi
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Prometheus
          type: prometheus
          url: http://prometheus-operated:9090
          access: proxy
          isDefault: true

# AlertManager
alertmanager:
  enabled: true
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'email'  # Configure conforme necessário
    receivers:
      - name: 'email'
        # email_configs:
        #   - to: 'admin@deeptreasury.com'

# Node Exporter (métricas do SO)
nodeExporter:
  enabled: true

# Kube State Metrics (métricas do Kubernetes)
kubeStateMetrics:
  enabled: true

# Prometheus Operator
prometheusOperator:
  enabled: true
```

### Passo 1.4: Instalar Chart

```bash
# Instalar kube-prometheus-stack
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --values kube-prometheus-values.yaml

# Aguardar instalação
# Pode levar 2-3 minutos
sleep 60

# Verificar pods
kubectl get pods -n monitoring

# Resultado esperado:
# NAME                                          READY   STATUS
# alertmanager-kube-prometheus-alertmanager-0   1/1     Running
# grafana-xxx                                   1/1     Running
# kube-prometheus-operator-xxx                  1/1     Running
# prometheus-kube-prometheus-prometheus-0       1/1     Running
# prometheus-node-exporter-xxx                  1/1     Running (um por node)
# kube-state-metrics-xxx                        1/1     Running
```

## Parte 2: Acessar Grafana

### Passo 2.1: Port-Forward para Grafana

```bash
# Port-forward da porta 3000
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80 &

# Agora acesse em seu browser
# http://localhost:3000
```

### Passo 2.2: Login Inicial

```
Username: admin
Password: (conforme configurado em valores, padrão: prom-operator ou seu customizado)
```

### Passo 2.3: Mudança Recomendada de Senha

1. Após fazer login
2. Menu (canto superior esquerdo)
3. Preferences → Change Password
4. Defina senha segura

## Parte 3: Acessar Prometheus

### Passo 3.1: Port-Forward para Prometheus

```bash
# Port-forward para Prometheus
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090 &

# Acesse em seu browser
# http://localhost:9090
```

### Passo 3.2: Explorar Métricas

No Prometheus:

1. Clique em **Graph**
2. Na caixa de texto, comece a digitar uma métrica:
   - `node_cpu_seconds_total` - CPU usage
   - `node_memory_MemFree_bytes` - Memory
   - `kube_pod_container_status_running` - Running pods
   - `rate(http_requests_total[5m])` - HTTP request rate

3. Clique em **Execute** para ver o gráfico

## Parte 4: Configurar Monitoring para DeepTreasury

### Passo 4.1: Habilitar Métricas na Aplicação

Sua aplicação DeepTreasury deve expor métricas Prometheus. No arquivo de configuração (Program.cs ou similar):

```csharp
// Adicionar Prometheus middleware
builder
    .UseOpenTelemetry()
    .WithTracing(tp =>
    {
        tp.AddAspNetCoreInstrumentation()
          .AddHttpClientInstrumentation()
          .AddOtlpExporter();
    })
    .WithMetrics(mp =>
    {
        mp.AddMeterListener(new PrometheusMetricsListener());
    });

// Ou usar Prometheus.NetStandard.Core
app.MapPrometheusScrapingEndpoint();  // Expõe /metrics
```

Resultado: Sua aplicação terá endpoint `/metrics` que Prometheus pode scrape.

### Passo 4.2: Criar ServiceMonitor para DeepTreasury

Crie um arquivo `servicemonitor-deeptreasury.yaml`:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: deeptreasury-monitor
  namespace: ts-dev
  labels:
    app: deeptreasury
spec:
  selector:
    matchLabels:
      app: deeptreasury
  endpoints:
    - port: metrics  # Nome do port no service
      interval: 30s
      path: /metrics
```

Aplicar:

```bash
kubectl apply -f servicemonitor-deeptreasury.yaml
```

### Passo 4.3: Garantir Service com Port "metrics"

Seu service DeepTreasury deve ter um port chamado "metrics":

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deeptreasury-service
  namespace: ts-dev
spec:
  selector:
    app: deeptreasury
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: metrics  # IMPORTANTE: adicionar esta porta
      port: 9090
      targetPort: 9090  # Onde a aplicação expõe /metrics
```

Aplicar:

```bash
kubectl apply -f service.yaml
```

### Passo 4.4: Validar Scraping

No Prometheus (http://localhost:9090):

1. Vá em **Status → Targets**
2. Procure por `deeptreasury-monitor`
3. Status deve ser "UP" (verde)
4. Se "DOWN", verifique logs e ServiceMonitor

## Parte 5: Criar Dashboards Customizados

### Passo 5.1: Importar Dashboard Pronto

No Grafana:

1. Menu superior esquerdo → **+** → **Import**
2. Digite o ID: `1860` (Node Exporter for Prometheus Dashboard)
3. Clique em **Load**
4. Selecione data source: `Prometheus`
5. Clique em **Import**

Outros dashboards úteis:
- `6417` - Kubernetes Cluster Monitoring
- `8588` - Kubernetes Deployment Statefulset Daemonset Metrics
- `12114` - System Exporter

### Passo 5.2: Criar Dashboard Customizado

No Grafana:

1. **+** → **Create → Dashboard**
2. **Add Panel**
3. Configure a query:
   ```
   rate(http_requests_total[5m])
   ```
4. Customize:
   - Title: "HTTP Request Rate"
   - Units: "Requests/sec"
   - Legend: "{{handler}}"
4. Clique em **Save**

## Parte 6: Configurar Alertas

### Passo 6.1: Criar Alertas via PrometheusRule

Crie um arquivo `prometheus-rules.yaml`:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: deeptreasury-alerts
  namespace: monitoring
spec:
  groups:
    - name: deeptreasury.rules
      interval: 30s
      rules:
        # Alerta 1: CPU alta
        - alert: HighCPUUsage
          expr: |
            (sum(rate(container_cpu_usage_seconds_total[5m])) by (pod_name)
             / sum(container_spec_cpu_quota) by (pod_name)) > 0.8
          for: 5m
          labels:
            severity: warning
            app: deeptreasury
          annotations:
            summary: "High CPU usage in {{ $labels.pod_name }}"
            description: "Pod {{ $labels.pod_name }} has CPU usage > 80% for 5 minutes"

        # Alerta 2: Memória alta
        - alert: HighMemoryUsage
          expr: |
            (container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.85
          for: 5m
          labels:
            severity: warning
            app: deeptreasury
          annotations:
            summary: "High memory usage in {{ $labels.pod_name }}"

        # Alerta 3: Taxa de erro alta
        - alert: HighErrorRate
          expr: |
            (sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)
             / sum(rate(http_requests_total[5m])) by (job)) > 0.05
          for: 5m
          labels:
            severity: critical
            app: deeptreasury
          annotations:
            summary: "High error rate in {{ $labels.job }}"
            description: "Error rate is {{ $value | humanizePercentage }} for 5 minutes"

        # Alerta 4: Conexão ao banco de dados
        - alert: DatabaseConnectionDown
          expr: |
            up{job="deeptreasury-db"} == 0
          for: 1m
          labels:
            severity: critical
            app: deeptreasury
          annotations:
            summary: "Database connection down"
            description: "Cannot reach database for the past minute"
```

Aplicar:

```bash
kubectl apply -f prometheus-rules.yaml

# Verificar
kubectl get prometheusrule -n monitoring
```

### Passo 6.2: Configurar Notificações de Alerta

No arquivo `kube-prometheus-values.yaml`, configure alertmanager:

```yaml
alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname', 'cluster']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 4h
      receiver: 'deeptreasury-team'
    receivers:
      - name: 'deeptreasury-team'
        email_configs:
          - to: 'admin@deeptreasury.com'
            from: 'alerts@deeptreasury.com'
            smarthost: 'smtp.gmail.com:587'
            auth_username: 'your-email@gmail.com'
            auth_password: 'your-app-password'
            require_tls: true
```

Reaplique:

```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --values kube-prometheus-values.yaml
```

## Parte 7: Acessar via Ingress (Opcional)

Se quiser acessar Prometheus e Grafana via HTTPS:

### Passo 7.1: Criar Ingress para Grafana

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - grafana.deeptreasury.com
      secretName: grafana-tls
  rules:
    - host: grafana.deeptreasury.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kube-prometheus-stack-grafana
                port:
                  number: 80
```

### Passo 7.2: DNS + Validação

```bash
# Atualize DNS para apontar a grafana.deeptreasury.com ao Load Balancer

# Teste após DNS propagar
curl https://grafana.deeptreasury.com
```

## Validação Completa

### Checklist Monitoring

- [ ] Todos os pods do `monitoring` namespace estão "Running"
- [ ] Prometheus consegue scrape do cluster
- [ ] Grafana acessa Prometheus como data source
- [ ] ServiceMonitor DeepTreasury está criado
- [ ] Prometheus consegue scrape de DeepTreasury (`/metrics`)
- [ ] Dashboards estão visíveis no Grafana
- [ ] Alertas foram criados

## Troubleshooting

### Problema: Prometheus não consegue scrape

**Debug:**
```bash
# Ver status dos targets
kubectl exec -it -n monitoring prometheus-kube-prometheus-prometheus-0 -- \
  wget -qO- http://localhost:9090/api/v1/targets

# Ver logs
kubectl logs -n monitoring prometheus-kube-prometheus-prometheus-0
```

### Problema: Grafana não consegue acessar Prometheus

**Debug:**
```bash
# Testar conectividade
kubectl exec -it -n monitoring kube-prometheus-stack-grafana-xxx -- \
  wget -qO- http://prometheus-operated:9090

# Ver logs Grafana
kubectl logs -n monitoring kube-prometheus-stack-grafana-xxx
```

### Problema: Alertas não disparam

**Debug:**
```bash
# Ver alertas definidos
kubectl get prometheusrule -n monitoring

# Validar sintaxe PromQL
# No Prometheus: http://localhost:9090/graph
# Digite sua expr e execute
```

### Problema: Storage cheio de Prometheus

**Solução:**
```bash
# Aumentar PVC
kubectl patch pvc prometheus-kube-prometheus-prometheus-db-prometheus-kube-prometheus-prometheus-0 \
  -n monitoring -p '{"spec":{"resources":{"requests":{"storage":"50Gi"}}}}'

# Ou ajustar retention
# Edite PrometheusSpec em values.yaml
# retention: 7d  # ao invés de 30d
```

## Próximos Passos

✅ Monitoring completo está configurado

**Próximos passos:**
1. **[`06-migrar-aplicacao.md`](./06-migrar-aplicacao.md)** - Migrar DeepTreasury
2. Configurar alertas customizados
3. Criar dashboards específicos para negócio

## Referências

- [Prometheus Operator](https://prometheus-operator.dev/)
- [kube-prometheus-stack Helm Chart](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
