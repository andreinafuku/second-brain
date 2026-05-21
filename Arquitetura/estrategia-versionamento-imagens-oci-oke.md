---
title: "Estrategia de Versionamento e Gestao de Imagens OCI OKE"
date: 2026-04-23
tags:
  - arquitetura
  - oci
  - oke
  - versionamento
  - status/rascunho
area: arquitetura
---

# Estratégia de Versionamento e Gestão de Imagens

Este documento apresenta as melhores práticas para versionamento e gestão de imagens Docker no Oracle Container Registry (OCIR) integrado com Kubernetes (OKE), focando em ambientes de desenvolvimento, homologação e produção.

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Resposta Rápida: Repositórios Separados ou Não?](#resposta-rápida-repositórios-separados-ou-não)
- [Estratégia Recomendada](#estratégia-recomendada)
- [Versionamento por Ambiente](#versionamento-por-ambiente)
- [Workflow Completo CI/CD](#workflow-completo-cicd)
- [Políticas de Retenção de Imagens](#políticas-de-retenção-de-imagens)
- [Pipeline GitHub Actions](#pipeline-github-actions)
- [Estrutura de Branches](#estrutura-de-branches)
- [Visualização Temporal das Tags](#visualização-temporal-das-tags)
- [Melhores Práticas](#melhores-práticas)

---

## 🎯 Visão Geral

### Contexto

No dia a dia de desenvolvimento:
- Desenvolvedores entregam mudanças via pull requests
- A cada PR, uma nova imagem deve ser gerada
- O Kubernetes deployment deve ser atualizado com essa imagem
- Em desenvolvimento, haverá múltiplas imagens geradas diariamente
- Quando pronto para produção, versões serão numeradas: 1.0.0, 1.1.0, 1.2.0, etc.

### Questões Principais

1. **Preciso de repositórios separados para dev e produção?**
2. **Como versionar imagens de desenvolvimento?**
3. **Como gerenciar o acúmulo de imagens?**
4. **Como promover uma imagem de dev → hom → prod?**

---

## 🎯 Resposta Rápida: Repositórios Separados ou Não?

### ❌ NÃO precisa de repositórios separados!

**Use o MESMO repositório com TAGS diferentes** para desenvolvimento, homologação e produção.

### Vantagens desta Abordagem

- ✅ Todas as versões em um só lugar
- ✅ Fácil rastreabilidade
- ✅ Simplicidade no gerenciamento
- ✅ Facilita rollback
- ✅ Políticas de retenção mais simples
- ✅ Reduz duplicação de camadas Docker (economia de armazenamento)

---

## 📦 Estratégia Recomendada

### Um Repositório, Múltiplas Tags

**Estrutura:**

```
sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end
├── dev-a3f5b2c          ← Desenvolvimento (hash do commit)
├── dev-b4d7e1a          ← Desenvolvimento (outro commit)
├── dev-c5e8f2b          ← Desenvolvimento (mais um commit)
├── hom-0.1.0-rc1        ← Release candidate em homologação
├── hom-0.1.0-rc2        ← Outra RC em homologação
├── 1.0.0                ← Produção (versão semântica)
├── 1.1.0                ← Produção (nova versão)
└── 1.1.1                ← Produção (patch)
```

### Exemplo Completo para Todas as Aplicações

```
sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/
│
├── front-end/
│   ├── dev-a3f5b2c
│   ├── dev-b4d7e1a
│   ├── 0.1.0-rc1
│   ├── 1.0.0
│   └── 1.1.0
│
├── back-end/
│   ├── dev-a3f5b2c
│   ├── dev-b4d7e1a
│   ├── 0.1.0-rc1
│   ├── 1.0.0
│   └── 1.1.0
│
└── ai-back-end/
    ├── dev-a3f5b2c
    ├── dev-b4d7e1a
    ├── 0.1.0-rc1
    ├── 1.0.0
    └── 1.1.0
```

---

## 🏷️ Versionamento por Ambiente

### 1. Desenvolvimento (dev-*)

**Formato:** `dev-<hash-git-curto>-<timestamp-opcional>`

**Exemplos:**
```
dev-a3f5b2c
dev-a3f5b2c-20250131
dev-a3f5b2c-20250131-143022
```

**Como gerar:**

```bash
# Opção 1: Apenas hash (recomendado)
TAG="dev-$(git rev-parse --short HEAD)"
# Resultado: dev-a3f5b2c

# Opção 2: Hash + data
TAG="dev-$(git rev-parse --short HEAD)-$(date +%Y%m%d)"
# Resultado: dev-a3f5b2c-20250131

# Opção 3: Hash + data + hora (para múltiplos builds no mesmo commit)
TAG="dev-$(git rev-parse --short HEAD)-$(date +%Y%m%d-%H%M%S)"
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/ai-agent:dev-$(git rev-parse --short HEAD)-$(Get-Date -Format 'yyyyMMdd_HHmmss') .
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/back-end:dev-$(git rev-parse --short HEAD)-$(Get-Date -Format 'yyyyMMdd_HHmmss') .
# Resultado: dev-a3f5b2c-20250131-143022
```

**Quando usar:**
- A cada merge de pull request na branch `develop` ou `main`
- A cada commit que você quer testar em dev
- Builds automáticos via CI/CD

**Vantagens:**
- ✅ Rastreabilidade total (hash = commit exato)
- ✅ Único por commit
- ✅ Facilita debug (correlação direta com código)
- ✅ Permite múltiplos builds do mesmo commit (com timestamp)

**PowerShell:**
```powershell
$GitHash = git rev-parse --short HEAD
$Tag = "dev-$GitHash"
# Resultado: dev-a3f5b2c
```

### 2. Homologação (hom-* ou rc-*)

**Formato:** `hom-<versao>-rc<numero>` ou `<versao>-rc<numero>`

**Exemplos:**
```
hom-0.1.0-rc1
hom-0.1.0-rc2
0.2.0-rc1
1.0.0-rc3
```

**Quando usar:**
- Feature está pronta para teste em homologação
- Release candidates antes de ir para produção
- Testes de aceitação do usuário (UAT)

**Como criar:**

```bash
# Re-tag de uma imagem dev aprovada
docker pull sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:dev-a3f5b2c
docker tag sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:dev-a3f5b2c \
           sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:0.1.0-rc1
docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:0.1.0-rc1
```

**Vantagens:**
- ✅ Indica versão planejada para produção
- ✅ Permite múltiplas RCs antes do release final
- ✅ Facilita comunicação com stakeholders

### 3. Produção (Versionamento Semântico)

**Formato:** `<MAJOR>.<MINOR>.<PATCH>`

Seguindo [Semantic Versioning 2.0.0](https://semver.org/):

**Exemplos:**
```
1.0.0      ← Primeira versão em produção
1.1.0      ← Nova feature (backward compatible)
1.1.1      ← Bug fix (backward compatible)
2.0.0      ← Breaking change
```

**Regras:**
- **MAJOR:** Incrementa quando há mudanças incompatíveis na API
- **MINOR:** Incrementa quando adiciona funcionalidade mantendo compatibilidade
- **PATCH:** Incrementa quando corrige bugs mantendo compatibilidade

**Quando usar:**
- Releases oficiais validados em homologação
- Deploys em ambiente de produção
- Versões estáveis e testadas

**Como criar:**

```bash
# Re-tag de uma RC aprovada
docker pull sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:0.1.0-rc2
docker tag sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:0.1.0-rc2 \
           sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:1.0.0
docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:1.0.0
```

**Vantagens:**
- ✅ Padrão da indústria
- ✅ Comunica claramente o tipo de mudança
- ✅ Facilita gerenciamento de dependências
- ✅ Suportado por ferramentas de automação

---

## 🔄 Workflow Completo CI/CD

### Cenário 1: Desenvolvedor Faz Pull Request

```
Desenvolvedor → PR → CI Build → Imagem dev-abc123 → Deploy em DEV
```

**1. Desenvolvedor cria PR:**

```bash
git checkout -b feature/nova-funcionalidade
# ... faz alterações ...
git commit -m "Adiciona nova funcionalidade"
git push origin feature/nova-funcionalidade
# Cria PR no GitHub/GitLab
```

**2. CI/CD executa automaticamente:**

```bash
# No pipeline CI/CD
COMMIT_HASH=$(git rev-parse --short HEAD)
TAG="dev-${COMMIT_HASH}"

# Build
docker build -t sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${TAG} .

# Push
docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${TAG}

# Atualizar deployment em DEV
kubectl set image deployment/front-end \
  front-end=sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${TAG} \
  -n ts-dev
```

**3. Resultado:**
- Nova imagem `dev-abc123` criada no OCIR
- Deploy automático no ambiente `ts-dev`
- Desenvolvedores podem testar imediatamente

### Cenário 2: Merge para Branch Principal (develop)

**Trigger:** Merge de PR aprovado

```bash
# Após aprovação do PR, ao fazer merge
COMMIT_HASH=$(git rev-parse --short HEAD)
TAG="dev-${COMMIT_HASH}"

# Build e push (mesma lógica do Cenário 1)
# Deploy automático em DEV
```

**Fluxo:**
```
PR aprovado → Merge → CI build → dev-xyz789 → Auto-deploy DEV
```

### Cenário 3: Preparar para Homologação

**Trigger:** Manual ou automático após testes em DEV

```bash
# Escolher a imagem dev que passou nos testes
SOURCE_TAG="dev-abc123"
RC_TAG="0.1.0-rc1"

# Re-tag da imagem dev escolhida
docker pull sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${SOURCE_TAG}
docker tag sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${SOURCE_TAG} \
           sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${RC_TAG}
docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${RC_TAG}

# Deploy em homologação
kubectl set image deployment/front-end \
  front-end=sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${RC_TAG} \
  -n ts-hom
```

**Fluxo:**
```
dev-abc123 (aprovado) → 0.1.0-rc1 → Deploy HOM → Testes UAT
```

### Cenário 4: Release para Produção

**Trigger:** Manual após validação completa em HOM

```bash
# Após validação bem-sucedida em homologação
RC_TAG="0.1.0-rc1"
PROD_TAG="1.0.0"

# Re-tag da RC aprovada
docker pull sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${RC_TAG}
docker tag sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${RC_TAG} \
           sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${PROD_TAG}
docker push sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${PROD_TAG}

# Criar Git tag
git tag -a v${PROD_TAG} -m "Release version ${PROD_TAG}"
git push origin v${PROD_TAG}

# Deploy em produção (quando existir o ambiente)
kubectl set image deployment/front-end \
  front-end=sa-saopaulo-1.ocir.io/xrtbrasilcloud3/treasury-360/front-end:${PROD_TAG} \
  -n production
```

**Fluxo:**
```
0.1.0-rc1 (validado) → 1.0.0 → Git tag v1.0.0 → Deploy PROD
```

### Diagrama de Fluxo Completo

```
┌─────────────┐
│ Desenvolve  │
│   Feature   │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────────┐
│   Cria PR   │────>│   CI Build   │
└─────────────┘     └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ dev-abc123   │
                    │ Push to OCIR │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Deploy DEV  │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Testes  DEV  │
                    └──────┬───────┘
                           │ Aprovado
                           ▼
                    ┌──────────────┐
                    │ 0.1.0-rc1    │
                    │ Re-tag       │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Deploy HOM  │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Testes  UAT  │
                    └──────┬───────┘
                           │ Aprovado
                           ▼
                    ┌──────────────┐
                    │   1.0.0      │
                    │ Release PROD │
                    └──────────────┘
```

---

## 📋 Políticas de Retenção de Imagens

### Problema

Com muitos PRs diários, centenas de imagens `dev-*` se acumulam, consumindo espaço de armazenamento e dificultando a navegação.

### Estratégia Recomendada

| Ambiente | Política de Retenção | Justificativa |
|----------|---------------------|---------------|
| **Desenvolvimento** | Manter últimas 30 imagens OU 30 dias | Alta rotatividade, imagens antigas raramente necessárias |
| **Homologação** | Manter últimas 10 RCs OU 90 dias | Médio volume, útil manter histórico recente |
| **Produção** | Manter todas OU últimas 20 versões | Baixo volume, importante para rollback e auditoria |

### Implementação Manual

```bash
# Listar imagens dev antigas (exemplo com OCI CLI)
oci artifacts container image list \
  --compartment-id <compartment-id> \
  --repository-id <repository-id> \
  --query "data.items[?starts_with(version, 'dev-')].{Version:version, Created:\"time-created\"}" \
  --all

# Deletar imagem específica
oci artifacts container image delete \
  --image-id <image-ocid> \
  --force
```

### Implementação Automatizada

**Script: `cleanup-old-dev-images.sh`**

```bash
#!/bin/bash
# cleanup-old-dev-images.sh
# Script para limpeza automática de imagens antigas de desenvolvimento

# Configurações
COMPARTMENT_ID="<seu-compartment-id>"
REPO_NAME="treasury-360/front-end"  # Ajustar para cada app
DAYS_TO_KEEP=30

echo "🧹 Iniciando limpeza de imagens antigas..."
echo "Repositório: $REPO_NAME"
echo "Mantendo imagens dos últimos $DAYS_TO_KEEP dias"
echo "================================================"

# Obter repository ID
REPO_ID=$(oci artifacts container repository list \
  --compartment-id $COMPARTMENT_ID \
  --display-name "$REPO_NAME" \
  --query 'data.items[0].id' \
  --raw-output)

if [ -z "$REPO_ID" ]; then
    echo "❌ Erro: Repositório não encontrado"
    exit 1
fi

# Calcular data limite
CUTOFF_DATE=$(date -d "-${DAYS_TO_KEEP} days" --iso-8601)
echo "Data de corte: $CUTOFF_DATE"
echo ""

# Listar e deletar imagens dev antigas
DELETED_COUNT=0

oci artifacts container image list \
  --repository-id $REPO_ID \
  --all \
  --query "data.items[?starts_with(version, 'dev-') && \"time-created\" < '${CUTOFF_DATE}'].{ID:id, Version:version, Created:\"time-created\"}" \
  --output json | jq -r '.[] | "\(.ID)|\(.Version)|\(.Created)"' | while IFS='|' read IMAGE_ID VERSION CREATED; do
    
    echo "🗑️  Deletando: $VERSION (criada em $CREATED)"
    
    oci artifacts container image delete \
      --image-id "$IMAGE_ID" \
      --force 2>/dev/null
    
    if [ $? -eq 0 ]; then
        ((DELETED_COUNT++))
        echo "   ✅ Deletada com sucesso"
    else
        echo "   ❌ Erro ao deletar"
    fi
    echo ""
done

echo "================================================"
echo "✅ Limpeza concluída!"
echo "Total de imagens removidas: $DELETED_COUNT"
```

**Agendar com cron (executar toda noite às 2h):**

```bash
# Editar crontab
crontab -e

# Adicionar linha
0 2 * * * /home/scripts/cleanup-old-dev-images.sh >> /var/log/cleanup-images.log 2>&1
```

### Política de Retenção no OCI Console

1. Acesse **Container Registry**
2. Selecione o repositório
3. **Actions** → **Edit**
4. Configure **Image retention policy**:
   - **Retention rule:** Keep last N images
   - **Number:** 30 (para dev) ou 10 (para hom)

---

## 🚀 Pipeline GitHub Actions

### Exemplo Completo

**`.github/workflows/build-and-deploy.yml`:**

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - develop
      - main
  pull_request:
    branches:
      - develop

env:
  REGISTRY: sa-saopaulo-1.ocir.io
  TENANCY: xrtbrasilcloud3
  PROJECT: rd-ocr

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [front-end, back-end, ai-back-end]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        fetch-depth: 0  # Para ter acesso ao histórico Git completo
    
    - name: Login to OCIR
      run: |
        echo "${{ secrets.OCIR_AUTH_TOKEN }}" | docker login ${{ env.REGISTRY }} \
          -u "${{ secrets.OCIR_USERNAME }}" \
          --password-stdin
    
    - name: Generate image tag
      id: tag
      run: |
        COMMIT_HASH=$(git rev-parse --short HEAD)
        TAG="dev-${COMMIT_HASH}"
        echo "tag=${TAG}" >> $GITHUB_OUTPUT
        echo "Generated tag: ${TAG}"
    
    - name: Build Docker image
      run: |
        IMAGE_PATH="${{ env.REGISTRY }}/${{ env.TENANCY }}/${{ env.PROJECT }}/${{ matrix.app }}:${{ steps.tag.outputs.tag }}"
        echo "Building: ${IMAGE_PATH}"
        docker build -t ${IMAGE_PATH} ./${{ matrix.app }}
    
    - name: Push Docker image
      run: |
        IMAGE_PATH="${{ env.REGISTRY }}/${{ env.TENANCY }}/${{ env.PROJECT }}/${{ matrix.app }}:${{ steps.tag.outputs.tag }}"
        echo "Pushing: ${IMAGE_PATH}"
        docker push ${IMAGE_PATH}
    
    - name: Deploy to DEV (only on push to develop)
      if: github.event_name == 'push' && github.ref == 'refs/heads/develop'
      run: |
        # Configurar kubectl
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
        export KUBECONFIG=./kubeconfig
        
        # Atualizar deployment
        kubectl set image deployment/${{ matrix.app }} \
          ${{ matrix.app }}=${{ env.REGISTRY }}/${{ env.TENANCY }}/${{ env.PROJECT }}/${{ matrix.app }}:${{ steps.tag.outputs.tag }} \
          -n ts-dev
        
        # Verificar rollout
        kubectl rollout status deployment/${{ matrix.app }} -n ts-dev --timeout=5m
    
    - name: Comment on PR with image tag
      if: github.event_name == 'pull_request'
      uses: actions/github-script@v6
      with:
        script: |
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: `✅ **${{ matrix.app }}** image built and pushed:\n\`${{ env.REGISTRY }}/${{ env.TENANCY }}/${{ env.PROJECT }}/${{ matrix.app }}:${{ steps.tag.outputs.tag }}\``
          })
```

### Secrets Necessários no GitHub

Configure em: **Settings** → **Secrets and variables** → **Actions**

| Secret | Descrição | Exemplo |
|--------|-----------|---------|
| `OCIR_USERNAME` | Username OCIR | `xrtbrasilcloud3/andre.inafuku@ext-xtpg.com.br` |
| `OCIR_AUTH_TOKEN` | Auth Token do OCI | `w)VqP4z{8kR>mN2xL5tH` |
| `KUBECONFIG` | Kubeconfig em base64 | `cat ~/.kube/config \| base64 -w 0` |

---

## 🗂️ Estrutura de Branches

### Git Flow Recomendado

```
main (ou master)
├── develop                    ← Branch de desenvolvimento
│   ├── feature/nova-func-1   ← Features em desenvolvimento
│   ├── feature/nova-func-2
│   └── bugfix/correcao-1
├── release/v1.0.0            ← Preparação para release
└── hotfix/urgente            ← Correções urgentes em produção
```

### Workflow Detalhado

**1. Desenvolvimento de Features:**

```bash
# Criar branch de feature a partir de develop
git checkout develop
git pull origin develop
git checkout -b feature/nova-funcionalidade

# Desenvolver, commitar
git add .
git commit -m "Implementa nova funcionalidade"

# Push e criar PR
git push origin feature/nova-funcionalidade
# Criar PR no GitHub/GitLab para merge em develop
```

**2. Merge em Develop:**

```bash
# Após aprovação do PR
# CI/CD automaticamente:
# - Faz build da imagem dev-<hash>
# - Push para OCIR
# - Deploy no namespace ts-dev
```

**3. Preparação para Release:**

```bash
# Quando develop estiver estável para release
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0

# Ajustes finais, bump de versão
# Criar RC
# CI/CD: build 1.0.0-rc1 → deploy ts-hom
```

**4. Release para Produção:**

```bash
# Após validação em homologação
git checkout main
git merge release/v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

# CI/CD: build 1.0.0 → deploy production
```

**5. Hotfix em Produção:**

```bash
# Para correção urgente em produção
git checkout main
git checkout -b hotfix/correcao-urgente

# Fazer correção
git commit -m "Fix: corrige bug crítico"

# Merge em main E develop
git checkout main
git merge hotfix/correcao-urgente
git tag -a v1.0.1 -m "Hotfix version 1.0.1"

git checkout develop
git merge hotfix/correcao-urgente

git push origin main develop --tags
```

---

## 📊 Visualização Temporal das Tags

### Exemplo ao Longo dos Meses

**Repositório: `treasury-360/front-end`**

#### Janeiro 2025
```
├── dev-a1b2c3d (PR #123 - Feature: Login - 31/01)
├── dev-a2b3c4d (PR #124 - Feature: Dashboard - 31/01)
├── dev-a3b4c5d (PR #125 - Bugfix: Validação - 31/01)
└── 0.1.0-rc1   (Release candidate para primeira versão)
```

#### Fevereiro 2025
```
├── dev-a4b5c6d (PR #126 - Feature: Relatórios - 01/02)
├── dev-a5b6c7d (PR #127 - Feature: Exportação - 01/02)
├── 0.1.0-rc2   (RC com correções de UAT)
├── 1.0.0       (🎉 Primeira versão em produção! - 05/02)
├── dev-a6b7c8d (PR #128 - Feature: Notificações - 10/02)
├── dev-a7b8c9d (PR #129 - Enhancement: UI - 15/02)
└── dev-a8b9c0d (PR #130 - Feature: API v2 - 20/02)
```

#### Março 2025
```
├── dev-a9b0c1d (PR #131 - Feature: Analytics - 01/03)
├── dev-b0c1d2e (PR #132 - Bugfix: Performance - 05/03)
├── 1.1.0-rc1   (RC com novas features)
├── 1.1.0       (Release com analytics - 10/03)
├── dev-b1c2d3e (PR #133 - Hotfix prep - 12/03)
└── 1.1.1       (Hotfix: correção crítica - 12/03)
```

### Timeline Visual

```
DEV    ─●─●─●─●─●─●─●─●─●─●─●─●─●─●→  (contínuo, muitos commits)
        │ │ │ │ │ │ │ │ │ │ │ │ │ │
HOM    ─┴─┴─┴─●───────●───────────●──→  (RCs selecionadas)
              │       │           │
PROD   ───────●───────●───────────●──→  (releases estáveis)
           1.0.0   1.1.0       1.1.1
```

---

## ✅ Melhores Práticas

### 1. Versionamento

#### ✅ FAÇA
- Use `dev-<hash-git>` para desenvolvimento (rastreável e único)
- Use `<versao>-rc<numero>` para release candidates (ex: `1.0.0-rc1`)
- Use versionamento semântico para produção (`MAJOR.MINOR.PATCH`)
- Mantenha correlação entre tags Git e versões de imagem
- Documente mudanças em CHANGELOG.md

#### ❌ NÃO FAÇA
- Nunca use tag `latest` em produção (impossível rastrear)
- Não reutilize tags (imutabilidade é fundamental)
- Evite tags genéricas como `stable`, `current`, `prod`
- Não versione manualmente sem automação

### 2. Repositórios

#### ✅ FAÇA
- Use um repositório por aplicação com múltiplas tags
- Organize com prefixos claros (ex: `treasury-360/front-end`)
- Configure políticas de acesso apropriadas
- Implemente políticas de retenção

#### ❌ NÃO FAÇA
- Não crie repositórios separados por ambiente
- Evite nomes ambíguos ou genéricos
- Não misture aplicações diferentes no mesmo repositório

### 3. Gestão de Imagens

#### ✅ FAÇA
- Implemente política de retenção automática para imagens dev
- Mantenha todas as versões de produção (ou últimas 20)
- Monitore uso de armazenamento
- Faça limpeza periódica de imagens não utilizadas
- Use multi-stage builds para reduzir tamanho das imagens

#### ❌ NÃO FAÇA
- Não acumule imagens dev indefinidamente
- Evite deletar versões de produção sem critério
- Não ignore alertas de armazenamento

### 4. CI/CD e Automação

#### ✅ FAÇA
- Automatize builds em cada PR/merge
- Implemente testes automatizados antes do build
- Configure notificações de sucesso/falha
- Use cache de layers Docker para agilizar builds
- Implemente scanning de segurança nas imagens
- Configure deploy automático apenas em DEV

#### ❌ NÃO FAÇA
- Não faça deploy automático em HOM ou PROD
- Evite pipelines sem validação/aprovação
- Não commite secrets ou credenciais
- Não pule testes para acelerar pipeline

### 5. Deploy e Promoção

#### ✅ FAÇA
- DEV: Deploy automático a cada merge
- HOM: Deploy manual ou com aprovação
- PROD: Sempre manual com múltiplas aprovações
- Implemente rollback automático em caso de falha
- Teste em ambientes inferiores primeiro
- Mantenha paridade entre ambientes

#### ❌ NÃO FAÇA
- Não pule ambientes (dev → prod diretamente)
- Evite deploys diretos em produção sem testes
- Não faça hotfixes sem processo adequado
- Não ignore health checks e readiness probes

### 6. Segurança

#### ✅ FAÇA
- Use secrets do Kubernetes para credenciais
- Implemente image scanning (vulnerabilidades)
- Rotacione auth tokens periodicamente
- Configure RBAC apropriadamente
- Use imagens base oficiais e atualizadas
- Implemente least privilege principle

#### ❌ NÃO FAÇA
- Nunca commite senhas ou tokens em código
- Não use imagens de fontes não confiáveis
- Evite executar containers como root
- Não exponha registries publicamente sem necessidade

### 7. Documentação

#### ✅ FAÇA
- Mantenha CHANGELOG.md atualizado
- Documente breaking changes
- Registre decisões de arquitetura (ADRs)
- Mantenha README com instruções de build
- Documente processo de release
- Crie runbooks para operações comuns

#### ❌ NÃO FAÇA
- Não deixe documentação desatualizada
- Evite assumir conhecimento implícito
- Não documente apenas no código

### 8. Monitoramento e Observabilidade

#### ✅ FAÇA
- Monitore uso de recursos do registry
- Configure alertas de falha em builds
- Rastreie métricas de deploy (MTTR, deploy frequency)
- Implemente logging centralizado
- Use tracing distribuído

#### ❌ NÃO FAÇA
- Não ignore métricas de performance
- Evite deploys sem capacidade de observar resultados
- Não deixe de monitorar ambientes de desenvolvimento

---

## 📚 Referências e Recursos

### Documentação Oficial

- [Oracle Container Registry Documentation](https://docs.oracle.com/en-us/iaas/Content/Registry/home.htm)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

### Ferramentas Recomendadas

- **CI/CD:** GitHub Actions, GitLab CI, Jenkins
- **Image Scanning:** Trivy, Clair, Anchore
- **Registry Management:** Harbor (se self-hosted)
- **Kubernetes Management:** ArgoCD, Flux (GitOps)

### Leitura Adicional

- [The Twelve-Factor App](https://12factor.net/)
- [GitOps Principles](https://www.gitops.tech/)
- [Container Best Practices](https://cloud.google.com/architecture/best-practices-for-building-containers)

---

## 📝 Checklist de Implementação

### Configuração Inicial
- [ ] Criar repositórios no OCIR (compartimento correto)
- [ ] Configurar secrets no GitHub/GitLab
- [ ] Configurar kubeconfig para CI/CD
- [ ] Definir estratégia de branches
- [ ] Criar pipeline inicial

### Para Cada Aplicação
- [ ] Dockerfile otimizado (multi-stage)
- [ ] Health check endpoints implementados
- [ ] Manifestos Kubernetes criados
- [ ] Variáveis de ambiente configuradas
- [ ] Scripts de build e push

### Operacional
- [ ] Política de retenção configurada
- [ ] Script de limpeza agendado
- [ ] Monitoramento configurado
- [ ] Processo de release documentado
- [ ] Runbooks criados

### Segurança
- [ ] Image scanning habilitado
- [ ] Secrets gerenciados corretamente
- [ ] RBAC configurado
- [ ] Network policies definidas
- [ ] Auditoria habilitada

---

**Última atualização:** Janeiro 2026  
**Versão do documento:** 1.0.0
