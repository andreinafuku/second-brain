# Configurar Cert-Manager para TLS/SSL Automático

## Introdução

O **Cert-Manager** automatiza a emissão, renovação e gerenciamento de certificados TLS/SSL em Kubernetes. Ele integra-se perfeitamente com NGINX Ingress para fornecer HTTPS seguro.

### Por Que Usar Cert-Manager?

- ✅ Automação completa de certificados (emissão, renovação, rotação)
- ✅ Integração com Let's Encrypt (grátis)
- ✅ Renovação automática 30 dias antes da expiração
- ✅ Suporte a múltiplos emissores (Issuers)
- ✅ Sem necessidade de gerenciamento manual de certificados

### Fluxo de Funcionamento

```
1. Você cria um Ingress com anotação cert-manager
2. Cert-Manager detecta automaticamente
3. Cert-Manager requisita certificado ao Let's Encrypt
4. Let's Encrypt valida domínio (HTTP-01 ou DNS-01)
5. Cert-Manager armazena certificado em Kubernetes Secret
6. NGINX usa Secret para servir HTTPS
7. Renovação automática acontece 30 dias antes de expiração
```

## Pré-Requisitos

- ✅ Cluster OKE criado
- ✅ NGINX Ingress Controller instalado (ver [`03-configurar-ingress-nginx.md`](./03-configurar-ingress-nginx.md))
- ✅ Domínio DNS configurado apontando para o Load Balancer
- ✅ kubectl configurado
- ✅ Helm instalado

## Parte 1: Instalar Cert-Manager

### Passo 1.1: Adicionar Repositório Helm

```bash
# Adicionar repositório do Cert-Manager
helm repo add jetstack https://charts.jetstack.io

# Atualizar cache
helm repo update

# Verificar
helm repo list | grep jetstack
```

### Passo 1.2: Criar Namespace

```bash
# Namespace dedicado para cert-manager
kubectl create namespace cert-manager

# Verificar
kubectl get namespace cert-manager
```

### Passo 1.3: Instalar CRDs (Custom Resource Definitions)

```bash
# CRDs são necessários para os tipos Certificate, ClusterIssuer, etc.
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml

# Verificar CRDs
kubectl get crds | grep cert-manager
# Resultado esperado: certificates.cert-manager.io, clusterissuers.cert-manager.io, etc.
```

### Passo 1.4: Instalar Chart Helm

```bash
# Instalar cert-manager
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.13.0

# Verificar pods
kubectl get pods -n cert-manager

# Resultado esperado:
# NAME                                      READY   STATUS
# cert-manager-xxx                          1/1     Running
# cert-manager-cainjector-xxx               1/1     Running
# cert-manager-webhook-xxx                  1/1     Running
```

> ⏱️ Aguarde 1-2 minutos para todos os pods ficarem "Running".

## Parte 2: Criar ClusterIssuer para Let's Encrypt

Um **ClusterIssuer** define como obter certificados. Você criará dois: um para staging (testes) e outro para produção.

### Passo 2.1: Criar ClusterIssuer Staging (Testes)

Crie um arquivo `cert-issuer-staging.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: admin@deeptreasury.com  # ALTERE PARA SEU EMAIL
    privateKeySecretRef:
      name: letsencrypt-staging-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

Aplicar:

```bash
kubectl apply -f cert-issuer-staging.yaml

# Verificar
kubectl get clusterissuer letsencrypt-staging
# Resultado esperado: Ready = True
```

### Passo 2.2: Criar ClusterIssuer Produção

Crie um arquivo `cert-issuer-prod.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@deeptreasury.com  # ALTERE PARA SEU EMAIL
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

Aplicar:

```bash
kubectl apply -f cert-issuer-prod.yaml

# Verificar
kubectl get clusterissuer letsencrypt-prod
# Resultado esperado: Ready = True
```

> ⏱️ Aguarde ~30 segundos para os ClusterIssuers ficarem "Ready".

## Parte 3: Atualizar Ingress com TLS

### Passo 3.1: Adicionar TLS ao Ingress DeepTreasury

Atualize o arquivo `ingress-treasure.yaml` que você criou em [`03-configurar-ingress-nginx.md`](./03-configurar-ingress-nginx.md):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: deeptreasury-ingress
  namespace: ts-dev
  annotations:
    kubernetes.io/ingress.class: nginx
    # Anotação cert-manager - usar issuer de produção
    cert-manager.io/cluster-issuer: letsencrypt-prod
    # NGINX anotações
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/enable-cors: "true"
spec:
  # Nova seção: TLS
  tls:
    - hosts:
        - api.deeptreasury.com  # ALTERE PARA SEU DOMÍNIO
      secretName: deeptreasury-tls  # Cert-Manager criará este secret automaticamente

  # Regras HTTP/HTTPS
  rules:
    - host: api.deeptreasury.com  # ALTERE PARA SEU DOMÍNIO
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

# Ver detalhes (incluindo TLS)
kubectl describe ingress deeptreasury-ingress -n ts-dev
```

> ⏱️ Cert-Manager detectará o Ingress e começará a processar o certificado (leva ~2-3 minutos).

### Passo 3.2: Acompanhar Progresso

```bash
# Ver eventos do Ingress
kubectl describe ingress deeptreasury-ingress -n ts-dev

# Esperado:
# Events:
#   Type    Reason                 Age   Message
#   ----    ------                 ---   -------
#   Normal  CreateCertificate      1m    Successfully created Certificate resource
#   Normal  CertificateIssued      2m    Certificate renewed
```

## Parte 4: Validar Certificado

### Passo 4.1: Verificar Secret do Certificado

```bash
# Listar secrets na namespace ts-dev
kubectl get secrets -n ts-dev

# Resultado esperado: secret "deeptreasury-tls" deve existir
# deeptreasury-tls    kubernetes.io/tls   2      5m

# Ver detalhes do secret
kubectl describe secret deeptreasury-tls -n ts-dev

# Ou decodificar o certificado
kubectl get secret deeptreasury-tls -n ts-dev -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text -noout
```

### Passo 4.2: Testar HTTPS

```bash
# Teste direto
curl https://api.deeptreasury.com/

# Com verbose para ver certificado
curl -v https://api.deeptreasury.com/

# Verificar expiração
openssl s_client -connect api.deeptreasury.com:443 -showcerts | grep -A 3 "Not After"

# Ou via curl
curl --insecure -v https://api.deeptreasury.com/ 2>&1 | grep "expire date"
```

> ✅ Se receber resposta 200 e certificado válido, está funcionando!

## Parte 5: Testes Completos

### Teste 1: HTTP Redireciona para HTTPS

```bash
# HTTP deve redirecionar para HTTPS
curl -i http://api.deeptreasury.com/

# Resultado esperado:
# HTTP/1.1 308 Permanent Redirect
# Location: https://api.deeptreasury.com/
```

### Teste 2: HTTPS com Certificado Let's Encrypt

```bash
# HTTPS deve funcionar com cert válido
curl -v https://api.deeptreasury.com/ 2>&1 | grep -A 5 "subject="

# Resultado esperado:
# subject=CN = api.deeptreasury.com
# issuer=C = US, O = Let's Encrypt, CN = R3
```

### Teste 3: Certificado Válido

```bash
# Verificar se certificado é válido
openssl s_client -connect api.deeptreasury.com:443 -showcerts </dev/null 2>/dev/null | \
  openssl x509 -noout -dates

# Resultado esperado:
# notBefore=Jan 15 10:30:00 2026 GMT
# notAfter=Apr 15 10:30:00 2026 GMT
```

## Parte 6: Configurar Renovação Automática

Cert-Manager renovará certificados automaticamente 30 dias antes da expiração. Você pode monitorar isso.

### Passo 6.1: Ver Status do Certificate

```bash
# Listar certificates
kubectl get certificates -n ts-dev

# Exemplo:
# NAME                 READY   SECRET                ISSUER             STATUS
# deeptreasury-tls     True    deeptreasury-tls      letsencrypt-prod   Certificate issued

# Ver detalhes
kubectl describe certificate deeptreasury-tls -n ts-dev
```

### Passo 6.2: Monitorar Logs de Renovação

```bash
# Logs do cert-manager
kubectl logs -n cert-manager -l app=cert-manager -f

# Procurar por mensagens de renovação (a cada 30 dias)
```

## Troubleshooting

### Problema: Certificate fica em "Not Ready"

**Debug:**
```bash
# Ver status do certificate
kubectl describe certificate deeptreasury-tls -n ts-dev

# Ver eventos
kubectl get events -n ts-dev

# Ver logs do cert-manager
kubectl logs -n cert-manager -l app=cert-manager

# Procurar por mensagens de erro
```

**Causas comuns:**
1. **Domínio não resolve:** Verifique se DNS está correto
   ```bash
   nslookup api.deeptreasury.com
   ```

2. **Validação ACME falhando:** Verifique se HTTP/80 é acessível
   ```bash
   curl -v http://api.deeptreasury.com/.well-known/acme-challenge/test
   ```

3. **ClusterIssuer não Ready:** Verifique email e configuração

### Problema: Certificado mostra erro de "Rate Limited"

**Causa:** Let's Encrypt tem limites de requisições.

**Solução:**
1. Use `letsencrypt-staging` para testes
2. Aguarde antes de tentar novamente (rate limit é por domínio)
3. Verifique: https://letsencrypt.org/docs/rate-limits/

### Problema: HTTPS retorna erro de certificado inválido

**Debug:**
```bash
# Verificar certificado no secret
kubectl get secret deeptreasury-tls -n ts-dev -o jsonpath='{.data.tls\.crt}' | \
  base64 -d | openssl x509 -text -noout

# Verificar se corresponde ao domínio
# Procurar por: "Subject Alternative Name: DNS:api.deeptreasury.com"
```

### Problema: Múltiplos certificados para mesmo domínio

**Solução:**
```bash
# Delete o secret inválido
kubectl delete secret deeptreasury-tls -n ts-dev

# Cert-Manager recriará automaticamente
# Aguarde ~2-3 minutos
```

## Migrar de Staging para Produção

Se você começou com `letsencrypt-staging`:

### Passo 1: Alterar ClusterIssuer no Ingress

```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod  # Alterar de staging
```

### Passo 2: Deletar Secret Antigo

```bash
kubectl delete secret deeptreasury-tls -n ts-dev
```

### Passo 3: Reapplicar Ingress

```bash
kubectl apply -f ingress-treasure.yaml

# Aguarde nova emissão (~2-3 minutos)
kubectl get certificate -n ts-dev
```

## Validação Completa

### Checklist Cert-Manager

- [ ] Pod `cert-manager` está "Running"
- [ ] Pod `cert-manager-webhook` está "Running"
- [ ] ClusterIssuer `letsencrypt-staging` está "Ready"
- [ ] ClusterIssuer `letsencrypt-prod` está "Ready"
- [ ] Certificate `deeptreasury-tls` está "Ready"
- [ ] Secret `deeptreasury-tls` existe
- [ ] curl HTTPS funciona sem erros de certificado
- [ ] `openssl s_client` mostra certificado Let's Encrypt válido

## Próximos Passos

✅ HTTPS/TLS automático está configurado

**Próximos passos:**
1. **[`05-configurar-monitoring.md`](./05-configurar-monitoring.md)** - Monitorar cluster e aplicação
2. **[`06-migrar-aplicacao.md`](./06-migrar-aplicacao.md)** - Migrar DeepTreasury para novo cluster

## Referências

- [Cert-Manager Official Documentation](https://cert-manager.io/)
- [Cert-Manager with NGINX](https://cert-manager.io/docs/tutorials/acme/nginx-ingress/)
- [Let's Encrypt Rate Limits](https://letsencrypt.org/docs/rate-limits/)
- [Let's Encrypt Best Practices](https://letsencrypt.org/docs/best-practices/)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
