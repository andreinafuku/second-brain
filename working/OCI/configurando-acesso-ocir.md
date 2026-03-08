# Configurando Acesso ao Oracle Container Registry (OCIR)

Este guia apresenta o processo completo para configurar o acesso ao Oracle Container Registry (OCIR), tanto localmente quanto no Kubernetes (OKE).

## 📋 Índice

- [Pré-requisitos](#pré-requisitos)
- [Passo 1: Criar Auth Token no OCI](#passo-1-criar-auth-token-no-oci)
- [Passo 2: Configurar Docker Localmente](#passo-2-configurar-docker-localmente)
- [Passo 3: Criar Kubernetes Secrets para OCIR](#passo-3-criar-kubernetes-secrets-para-ocir)
- [Passo 4: Verificar Configuração](#passo-4-verificar-configuração)
- [Passo 5: Usar o Secret nos Deployments](#passo-5-usar-o-secret-nos-deployments)
- [Descobrir Informações do Tenancy](#descobrir-informações-do-tenancy)
- [Script de Automação](#script-de-automação)
- [Troubleshooting](#troubleshooting)
- [Checklist Final](#checklist-final)

---

## 📋 Pré-requisitos

Você precisa ter em mãos:

- ✅ Tenancy namespace do OCI
- ✅ Região do OCI (ex: sa-saopaulo-1)
- ✅ Usuário OCI (seu username)
- ✅ Auth Token (vamos criar agora)

---

## 🔑 Passo 1: Criar Auth Token no OCI

### 1.1. Acessar o Console OCI

1. Faça login no [Oracle Cloud Console](https://cloud.oracle.com)
2. Clique no ícone do **perfil** (canto superior direito)
3. Clique no seu **username**

### 1.2. Gerar Auth Token

1. No menu lateral esquerdo, clique em **Auth Tokens**
2. Clique em **Generate Token**
3. Dê um nome descritivo: `ocir-kubernetes-token`
4. Clique em **Generate Token**
5. ⚠️ **IMPORTANTE:** Copie o token gerado e salve em local seguro (ele aparece apenas uma vez!)

Exemplo de token:
```
w)VqP4z{8kR>mN2xL5tH
```

---

## 🐳 Passo 2: Configurar Docker Localmente

### 2.1. Identificar seu Endpoint OCIR

O formato do endpoint OCIR é:
```
<region-key>.ocir.io
```

**Principais regiões:**
- São Paulo: `gru.ocir.io`
- Vinhedo: `vcp.ocir.io`
- Ashburn: `iad.ocir.io`
- Phoenix: `phx.ocir.io`

[Lista completa de region keys](https://docs.oracle.com/en-us/iaas/Content/Registry/Concepts/registryprerequisites.htm#regional-availability)

### 2.2. Fazer Login no Docker

O formato do username para OCIR é:
```
<tenancy-namespace>/<oci-username>
```

**Exemplo de comando:**
```bash
docker login gru.ocir.io
```

Quando solicitado:
- **Username:** `<tenancy-namespace>/<seu-usuario-oci>`
  - Exemplo: `grxyz123/oracleidentitycloudservice/joao.silva@empresa.com`
- **Password:** Cole o Auth Token gerado anteriormente

**Exemplo completo:**
```bash
docker login gru.ocir.io
# Username: grxyz123/oracleidentitycloudservice/joao.silva@empresa.com
# Password: w)VqP4z{8kR>mN2xL5tH
```
**Como eu fiz:**

```bash
docker login sa-saopaulo-1.ocir.io
# Username: xrtbrasilcloud3/andre.inafuku@ext-xtpg.com.br
# Password: colei_aqui_token_gerado
```

### 2.3. Verificar Configuração

Suas credenciais ficam salvas em:
```bash
cat ~/.docker/config.json
```

Você verá algo assim:
```json
{
  "auths": {
    "gru.ocir.io": {
      "auth": "Z3J4eXoxMjMvb3JhY2xlaWRlbnRpdHljbG91ZHNlcnZpY2Uvam9hby5zaWx2YUBlbXByZXNhLmNvbTp3KVZxUDR6ezhrUj5tTjJ4TDV0SA=="
    }
  }
}
```

O meu ficou assim:

```json
{
  "auths": {
    "acrxtpg.azurecr.io": {},
    "https://index.docker.io/v1/": {},
    "sa-saopaulo-1.ocir.io": {}
  },
 "credsStore": "desktop",
 "currentContext": "desktop-linux"
}
```
---

## ☸️ Passo 3: Criar Kubernetes Secrets para OCIR

### 3.1. Método Direto (Usando suas credenciais Docker)

```bash
# Para namespace ts-dev
kubectl create secret docker-registry ocir-secret \
  --docker-server=gru.ocir.io \
  --docker-username='<tenancy-namespace>/<oci-username>' \
  --docker-password='<auth-token>' \
  --docker-email=<seu-email> \
  -n ts-dev

# Para namespace ts-hom
kubectl create secret docker-registry ocir-secret \
  --docker-server=gru.ocir.io \
  --docker-username='<tenancy-namespace>/<oci-username>' \
  --docker-password='<auth-token>' \
  --docker-email=<seu-email> \
  -n ts-hom
```

**Exemplo prático:**
```bash
kubectl create secret docker-registry ocir-secret \
  --docker-server=gru.ocir.io \
  --docker-username='grxyz123/oracleidentitycloudservice/joao.silva@empresa.com' \
  --docker-password='w)VqP4z{8kR>mN2xL5tH' \
  --docker-email=joao.silva@empresa.com \
  -n ts-dev
```
**Como eu fiz:**
```bash
kubectl create secret docker-registry ocir-secret \
  --docker-server=sa-saopaulo-1.ocir.io \
  --docker-username='xrtbrasilcloud3/andre.inafuku@ext-xtpg.com.br' \
  --docker-password='coloquei_meu_token' \
  --docker-email=andre.inafuku@ext-xtpg.com.br \
  -n ts-dev
```


### 3.2. Método Alternativo (Usando config.json existente)

Se você já fez login com Docker:

```bash
kubectl create secret generic ocir-secret \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson \
  -n ts-dev

kubectl create secret generic ocir-secret \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson \
  -n ts-hom
```

### 3.3. Método YAML (Recomendado para versionamento)

**⚠️ IMPORTANTE:** Nunca commite secrets com valores reais no Git!

Crie um arquivo template:

**`k8s/secrets/ocir-secret-template.yaml`:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ocir-secret
  namespace: ts-dev
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <BASE64_ENCODED_DOCKER_CONFIG>
```

**Gerar o base64:**
```bash
# Opção 1: A partir do .docker/config.json
cat ~/.docker/config.json | base64 -w 0

# Opção 2: Criar manualmente o JSON e encodar
echo -n '{"auths":{"gru.ocir.io":{"username":"<tenancy>/<user>","password":"<token>","auth":"<base64-user:pass>"}}}' | base64 -w 0
```

**Ou criar o auth em base64:**
```bash
echo -n '<tenancy-namespace>/<username>:<auth-token>' | base64 -w 0
```

**Aplicar:**
```bash
kubectl apply -f k8s/secrets/ocir-secret.yaml
```

---

## ✅ Passo 4: Verificar Configuração

### 4.1. Verificar Secrets Criados

```bash
# Listar secrets
kubectl get secrets -n ts-dev
kubectl get secrets -n ts-hom

# Ver detalhes (sem mostrar valores)
kubectl describe secret ocir-secret -n ts-dev
```

### 4.2. Testar Pull de Imagem

Criar imagem

Fazer pull de uma imagem
docker pull nginx:alpine

Criar tag
docker tag nginx:alpine sa-saopaulo-1.ocir.io/xrtbrasilcloud3/xrt-interno/rd-ocr/nginx:latest


push

docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/xrt-interno/rd-ocr/nginx:latest

Crie um pod de teste:

**`test-ocir-pull.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ocir
  namespace: ts-dev
spec:
  containers:
  - name: nginx
    image: sa-saopaulo-1.ocir.io/xrtbrasilcloud3/nginx:latest
  imagePullSecrets:
  - name: ocir-secret
```

```bash
kubectl apply -f test-ocir-pull.yaml -n ts-dev
kubectl get pod test-ocir -n ts-dev
kubectl describe pod test-ocir -n ts-dev
```

Se aparecer `ImagePullBackOff`, verifique os logs:
```bash
kubectl describe pod test-ocir -n ts-dev | grep -A 10 Events
```

**Limpar teste:**
```bash
kubectl delete pod test-ocir -n ts-dev
```

---

## 📦 Passo 5: Usar o Secret nos Deployments

Adicione `imagePullSecrets` em todos os seus Deployments:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ts-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      imagePullSecrets:
      - name: ocir-secret  # ← IMPORTANTE!
      containers:
      - name: frontend
        image: gru.ocir.io/<tenancy-namespace>/frontend:dev-v1
        ports:
        - containerPort: 80
```

---

## 🔍 Descobrir Informações do Tenancy

### Tenancy Namespace

```bash
# Via OCI CLI
oci os ns get

# Ou no Console OCI:
# Profile → Tenancy → Object Storage Namespace
```

### Region Key

No Console OCI, veja a região no canto superior direito.

Mapeamento comum:
- `Brazil East (Sao Paulo)` → `gru`
- `Brazil Southeast (Vinhedo)` → `vcp`

---

## 📝 Script de Automação

Crie um script para facilitar:

**`scripts/setup-ocir-secrets.sh`:**
```bash
#!/bin/bash

# Configurações
REGION_KEY="gru"
TENANCY_NAMESPACE="grxyz123"
OCI_USERNAME="oracleidentitycloudservice/joao.silva@empresa.com"
EMAIL="joao.silva@empresa.com"

echo "⚠️  Cole o Auth Token (não será exibido):"
read -s AUTH_TOKEN

# Criar secrets nos namespaces
for NS in ts-dev ts-hom; do
  echo "Criando secret no namespace $NS..."
  kubectl create secret docker-registry ocir-secret \
    --docker-server=${REGION_KEY}.ocir.io \
    --docker-username="${TENANCY_NAMESPACE}/${OCI_USERNAME}" \
    --docker-password="${AUTH_TOKEN}" \
    --docker-email=${EMAIL} \
    -n $NS \
    --dry-run=client -o yaml | kubectl apply -f -
done

echo "✅ Secrets criados com sucesso!"
```

**Executar:**
```bash
chmod +x scripts/setup-ocir-secrets.sh
./scripts/setup-ocir-secrets.sh
```

---

## 🚨 Troubleshooting

### Erro: "unauthorized: authentication required"

- Verifique se o Auth Token está correto
- Verifique o formato do username: `<tenancy>/<user>`
- Certifique-se de que o usuário tem permissões no OCIR

### Erro: "ImagePullBackOff"

```bash
# Ver logs detalhados
kubectl describe pod <pod-name> -n ts-dev

# Verificar se o secret existe
kubectl get secret ocir-secret -n ts-dev

# Testar pull manual
docker pull gru.ocir.io/<tenancy>/image:tag
```

### Como atualizar o secret

```bash
# Deletar
kubectl delete secret ocir-secret -n ts-dev

# Recriar
kubectl create secret docker-registry ocir-secret ...
```

---

## ✅ Checklist Final

- [ ] Auth Token criado no OCI Console
- [ ] Docker login funcionando localmente (`docker login gru.ocir.io`)
- [ ] Secret `ocir-secret` criado no namespace `ts-dev`
- [ ] Secret `ocir-secret` criado no namespace `ts-hom`
- [ ] Secrets verificados com `kubectl get secrets`
- [ ] Auth Token salvo em local seguro (gerenciador de senhas)

---

## 📚 Referências

- [Oracle Cloud Infrastructure Registry Documentation](https://docs.oracle.com/en-us/iaas/Content/Registry/home.htm)
- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Docker Login Documentation](https://docs.docker.com/engine/reference/commandline/login/)

---

**Última atualização:** Janeiro 2026