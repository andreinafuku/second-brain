# Guia de Setup WSL2 - IntelligencePortal Development Environment

**Autor:** Minoru  
**Data:** 26 de fevereiro de 2025  
**Objetivo:** Configurar ambiente híbrido Windows + Linux para desenvolvimento do IntelligencePortal

---

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Instalação do WSL2](#instalação-do-wsl2)
3. [Configuração do Ubuntu](#configuração-do-ubuntu)
4. [Python e FastAPI](#python-e-fastapi)
5. [Docker e Kubernetes](#docker-e-kubernetes)
6. [Oracle Client](#oracle-client)
7. [VS Code Integration](#vs-code-integration)
8. [Otimizações de Performance](#otimizações-de-performance)
9. [Backup e Manutenção](#backup-e-manutenção)
10. [Troubleshooting](#troubleshooting)

---

## Pré-requisitos

### Verificar versão do Windows

O WSL2 requer Windows 10 versão 2004+ ou Windows 11.

```powershell
# Verificar versão do Windows (PowerShell como Admin)
winver
```

### Habilitar recursos necessários

```powershell
# Executar no PowerShell como Administrador
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**⚠️ REINICIE O COMPUTADOR após executar esses comandos**

---

## Instalação do WSL2

### 1. Atualizar o componente WSL

```powershell
# PowerShell como Admin
wsl --update
```

### 2. Definir WSL2 como padrão

```powershell
wsl --set-default-version 2
```

### 3. Instalar Ubuntu 22.04 LTS

```powershell
# Listar distribuições disponíveis
wsl --list --online

# Instalar Ubuntu 22.04 (mesma versão do OKE)
wsl --install -d Ubuntu-22.04
```

Você será solicitado a criar um usuário e senha. Recomendo usar o mesmo nome de usuário que você usa no Windows para facilitar.

### 4. Verificar instalação

```powershell
# Listar distribuições instaladas
wsl --list --verbose

# Deve mostrar:
# NAME            STATE           VERSION
# Ubuntu-22.04    Running         2
```

---

## Configuração do Ubuntu

### 1. Atualizar sistema

```bash
# Dentro do WSL2 (terminal Ubuntu)
sudo apt update && sudo apt upgrade -y
```

### 2. Instalar ferramentas essenciais

```bash
# Build tools e utilitários
sudo apt install -y \
    build-essential \
    curl \
    wget \
    git \
    vim \
    nano \
    unzip \
    ca-certificates \
    gnupg \
    lsb-release \
    software-properties-common
```

### 3. Configurar Git

```bash
# Configurar identidade Git
git config --global user.name "Minoru"
git config --global user.email "seu-email@empresa.com"

# Configurar credenciais (integração com Windows)
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

### 4. Criar estrutura de diretórios

```bash
# Criar estrutura de projetos
mkdir -p ~/projects/intelligenceportal
mkdir -p ~/projects/fraud-detection
mkdir -p ~/tools
mkdir -p ~/scripts

# Criar link simbólico para facilitar acesso aos documentos Windows
ln -s /mnt/c/Users/Minoru/Documents ~/windows-docs
```

---

## Python e FastAPI

### 1. Instalar Python 3.11

```bash
# Adicionar repositório deadsnakes
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update

# Instalar Python 3.11 e ferramentas
sudo apt install -y \
    python3.11 \
    python3.11-venv \
    python3.11-dev \
    python3-pip \
    python3.11-distutils

# Definir Python 3.11 como padrão
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
sudo update-alternatives --config python3
```

### 2. Atualizar pip e instalar ferramentas

```bash
# Atualizar pip
python3 -m pip install --upgrade pip

# Instalar ferramentas de desenvolvimento
pip install --user \
    pipenv \
    poetry \
    black \
    flake8 \
    pylint \
    mypy \
    ipython
```

### 3. Setup do projeto Fraud Detection

```bash
cd ~/projects/fraud-detection

# Criar ambiente virtual
python3 -m venv venv

# Ativar ambiente virtual
source venv/bin/activate

# Instalar dependências do projeto
pip install fastapi uvicorn[standard] \
    langchain langchain-community \
    python-dotenv \
    pydantic pydantic-settings \
    httpx \
    pytest pytest-asyncio \
    cx_Oracle
```

### 4. Criar arquivo de requirements

```bash
# Criar requirements.txt
cat > ~/projects/fraud-detection/requirements.txt << 'EOF'
# FastAPI e servidor
fastapi==0.109.0
uvicorn[standard]==0.27.0
pydantic==2.5.3
pydantic-settings==2.1.0

# AI e LangChain
langchain==0.1.4
langchain-community==0.0.16

# Database
cx_Oracle==8.3.0
sqlalchemy==2.0.25

# Utilities
python-dotenv==1.0.0
httpx==0.26.0

# Development
pytest==7.4.4
pytest-asyncio==0.23.3
black==24.1.0
flake8==7.0.0
mypy==1.8.0
EOF
```

### 5. Criar estrutura do projeto FastAPI

```bash
cd ~/projects/fraud-detection

# Criar estrutura de diretórios
mkdir -p app/{api,core,models,services,schemas}
touch app/__init__.py
touch app/main.py
touch app/api/__init__.py
touch app/core/__init__.py
touch app/models/__init__.py
touch app/services/__init__.py
touch app/schemas/__init__.py

# Criar .env de exemplo
cat > .env.example << 'EOF'
# Oracle Database
ORACLE_USER=your_user
ORACLE_PASSWORD=your_password
ORACLE_DSN=your_host:1521/your_service

# OCI GenAI
OCI_COMPARTMENT_ID=your_compartment_id
OCI_ENDPOINT=https://inference.generativeai.us-chicago-1.oci.oraclecloud.com

# Application
APP_NAME=fraud-detection-api
APP_VERSION=1.0.0
DEBUG=True
EOF
```

---

## Docker e Kubernetes

### 1. Instalar Docker Desktop (Windows)

1. Baixe o Docker Desktop do site oficial: https://www.docker.com/products/docker-desktop
2. Durante a instalação, marque a opção **"Use WSL 2 instead of Hyper-V"**
3. Após instalação, abra Docker Desktop
4. Vá em Settings → General → marque **"Use the WSL 2 based engine"**
5. Vá em Settings → Resources → WSL Integration
6. Habilite integração com Ubuntu-22.04

### 2. Verificar instalação no WSL2

```bash
# Dentro do WSL2
docker --version
docker-compose --version

# Testar Docker
docker run hello-world
```

### 3. Instalar kubectl

```bash
# Baixar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Tornar executável
chmod +x kubectl

# Mover para bin
sudo mv kubectl /usr/local/bin/

# Verificar instalação
kubectl version --client
```

### 4. Instalar Minikube (Kubernetes local)

```bash
# Baixar Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Instalar
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verificar instalação
minikube version
```

### 5. Configurar Minikube com Docker

```bash
# Iniciar Minikube usando Docker driver
minikube start --driver=docker --memory=4096 --cpus=2

# Habilitar addons úteis
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons enable ingress

# Verificar status
minikube status
kubectl get nodes
```

### 6. Configurar acesso ao OKE

```bash
# Criar diretório para kubeconfig
mkdir -p ~/.kube

# Copiar kubeconfig do OKE (você precisará obter isso do OCI Console)
# OCI Console → Developer Services → Kubernetes Clusters → seu cluster → Access Cluster
# Copiar o conteúdo do kubeconfig para ~/.kube/config
```

### 7. Configurar contexts do kubectl

```bash
# Listar contexts disponíveis
kubectl config get-contexts

# Renomear contexts para facilitar
kubectl config rename-context minikube local-dev
kubectl config rename-context <context-oke> oke-ipt-dev

# Alternar entre contexts
kubectl config use-context local-dev    # Ambiente local
kubectl config use-context oke-ipt-dev  # OKE desenvolvimento
```

### 8. Criar namespace local para testes

```bash
# Criar namespace similar ao OKE
kubectl create namespace ipt-dev

# Definir namespace padrão
kubectl config set-context --current --namespace=ipt-dev
```

---

## Oracle Client

### 1. Baixar Oracle Instant Client

```bash
# Criar diretório para Oracle
mkdir -p ~/tools/oracle
cd ~/tools/oracle

# Baixar Oracle Instant Client Basic e SDK
# Visite: https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html
# Baixe a versão 21.x (compatível com Oracle 21c)

# Exemplo com wget (substitua pela versão mais recente)
wget https://download.oracle.com/otn_software/linux/instantclient/2115000/instantclient-basic-linux.x64-21.15.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/2115000/instantclient-sdk-linux.x64-21.15.0.0.0dbru.zip
```

### 2. Instalar Oracle Instant Client

```bash
# Instalar dependências
sudo apt install -y libaio1

# Extrair arquivos
unzip instantclient-basic-linux.x64-21.15.0.0.0dbru.zip
unzip instantclient-sdk-linux.x64-21.15.0.0.0dbru.zip

# Criar links simbólicos
cd instantclient_21_15
sudo mkdir -p /opt/oracle
sudo mv ~/tools/oracle/instantclient_21_15 /opt/oracle/

# Criar link do cliente
cd /opt/oracle/instantclient_21_15
sudo ln -s libclntsh.so.21.1 libclntsh.so
sudo ln -s libocci.so.21.1 libocci.so
```

### 3. Configurar variáveis de ambiente

```bash
# Adicionar ao ~/.bashrc
cat >> ~/.bashrc << 'EOF'

# Oracle Instant Client
export ORACLE_HOME=/opt/oracle/instantclient_21_15
export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME:$PATH

EOF

# Recarregar bashrc
source ~/.bashrc
```

### 4. Instalar cx_Oracle

```bash
# Ativar ambiente virtual do projeto
cd ~/projects/fraud-detection
source venv/bin/activate

# Instalar cx_Oracle
pip install cx_Oracle

# Ou usar python-oracledb (nova biblioteca, não requer Instant Client)
pip install oracledb
```

### 5. Testar conexão

```bash
# Criar script de teste
cat > ~/projects/fraud-detection/test_oracle.py << 'EOF'
import cx_Oracle
import os

# Configurar conexão
dsn = cx_Oracle.makedsn(
    "your-oracle-host",
    1521,
    service_name="your-service-name"
)

try:
    connection = cx_Oracle.connect(
        user="your_user",
        password="your_password",
        dsn=dsn
    )
    
    print("Conexão estabelecida com sucesso!")
    
    # Testar query simples
    cursor = connection.cursor()
    cursor.execute("SELECT BANNER FROM V$VERSION")
    for row in cursor:
        print(row[0])
    
    cursor.close()
    connection.close()
    
except cx_Oracle.Error as error:
    print(f"Erro ao conectar: {error}")
EOF

# Executar teste
python test_oracle.py
```

---

## VS Code Integration

### 1. Instalar VS Code (Windows)

Se ainda não tiver instalado, baixe de: https://code.visualstudio.com/

### 2. Instalar extensões essenciais

No VS Code, instale as seguintes extensões:

- **Remote - WSL** (ms-vscode-remote.remote-wsl)
- **Remote Development** (ms-vscode-remote.vscode-remote-extensionpack)
- **Python** (ms-python.python)
- **Pylance** (ms-python.vscode-pylance)
- **Docker** (ms-azuretools.vscode-docker)
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools)
- **YAML** (redhat.vscode-yaml)
- **GitLens** (eamodio.gitlens)

### 3. Abrir projeto no WSL2

```bash
# Método 1: Do terminal WSL2
cd ~/projects/fraud-detection
code .

# Método 2: Do VS Code Windows
# Ctrl+Shift+P → Remote-WSL: Open Folder in WSL
# Navegar até ~/projects/fraud-detection
```

### 4. Configurar settings do projeto

Crie `.vscode/settings.json` no projeto:

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/venv/bin/python",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.linting.flake8Enabled": true,
    "python.formatting.provider": "black",
    "python.formatting.blackArgs": ["--line-length", "100"],
    "editor.formatOnSave": true,
    "editor.rulers": [100],
    "files.exclude": {
        "**/__pycache__": true,
        "**/*.pyc": true
    },
    "python.testing.pytestEnabled": true,
    "python.testing.unittestEnabled": false,
    "python.analysis.typeCheckingMode": "basic"
}
```

### 5. Configurar launch.json para debugging

Crie `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: FastAPI",
            "type": "python",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "app.main:app",
                "--reload",
                "--host", "0.0.0.0",
                "--port", "8000"
            ],
            "jinja": true,
            "justMyCode": true,
            "env": {
                "PYTHONPATH": "${workspaceFolder}"
            }
        },
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": true
        }
    ]
}
```

### 6. Terminal integrado

O terminal integrado do VS Code automaticamente abrirá no WSL2 quando você abrir um projeto WSL. Você pode:

- Criar múltiplos terminais (Ctrl+Shift+`)
- Alternar entre PowerShell Windows e bash WSL2
- Executar comandos diretamente no ambiente Linux

---

## Otimizações de Performance

### 1. Configurar .wslconfig

Crie arquivo `.wslconfig` no Windows em `C:\Users\Minoru\.wslconfig`:

```ini
[wsl2]
# Limitar memória (metade da RAM total)
memory=16GB

# Limitar CPUs (deixar 2 cores para Windows)
processors=6

# Swap
swap=8GB

# Localhost forwarding
localhostForwarding=true

# Habilitar systemd (útil para alguns serviços)
[boot]
systemd=true
```

### 2. Configurar DNS personalizado (se necessário)

Crie `/etc/wsl.conf` no WSL2:

```bash
sudo nano /etc/wsl.conf
```

Adicione:

```ini
[network]
generateResolvConf = false

[boot]
systemd=true

[interop]
enabled=true
appendWindowsPath=true
```

Depois, configure DNS manualmente:

```bash
sudo rm /etc/resolv.conf
sudo nano /etc/resolv.conf
```

Adicione:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### 3. Otimizar Docker

No Docker Desktop (Windows):

1. Settings → Resources → Advanced
2. CPUs: 4-6 cores
3. Memory: 8GB
4. Swap: 2GB
5. Disk image size: 100GB+

### 4. Reiniciar WSL2 após mudanças

```powershell
# PowerShell como Admin
wsl --shutdown

# Aguardar 8 segundos
Start-Sleep -Seconds 8

# Iniciar novamente
wsl
```

---

## Backup e Manutenção

### 1. Exportar distribuição WSL2

```powershell
# PowerShell
# Criar backup completo
wsl --export Ubuntu-22.04 C:\Backups\ubuntu-22.04-backup.tar

# Restaurar de backup (se necessário)
wsl --import Ubuntu-22.04-Restored C:\WSL\Ubuntu-Restored C:\Backups\ubuntu-22.04-backup.tar
```

### 2. Backup incremental de projetos

```bash
# Script de backup incremental
cat > ~/scripts/backup-projects.sh << 'EOF'
#!/bin/bash

BACKUP_DIR="/mnt/c/Users/Minoru/Backups/WSL2-Projects"
DATE=$(date +%Y%m%d-%H%M%S)

mkdir -p "$BACKUP_DIR"

# Backup dos projetos
tar -czf "$BACKUP_DIR/projects-$DATE.tar.gz" ~/projects/

# Manter apenas últimos 5 backups
cd "$BACKUP_DIR"
ls -t projects-*.tar.gz | tail -n +6 | xargs -r rm

echo "Backup completed: projects-$DATE.tar.gz"
EOF

chmod +x ~/scripts/backup-projects.sh
```

### 3. Limpeza de espaço

```bash
# Script de limpeza
cat > ~/scripts/cleanup.sh << 'EOF'
#!/bin/bash

# Limpar cache do apt
sudo apt clean
sudo apt autoclean
sudo apt autoremove -y

# Limpar cache do pip
pip cache purge

# Limpar Docker
docker system prune -af --volumes

# Limpar logs antigos
sudo journalctl --vacuum-time=7d

echo "Cleanup completed!"
EOF

chmod +x ~/scripts/cleanup.sh
```

### 4. Atualização automática semanal

```bash
# Script de atualização
cat > ~/scripts/update-system.sh << 'EOF'
#!/bin/bash

echo "Atualizando sistema Ubuntu..."
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y

echo "Atualizando pip packages..."
pip list --outdated --format=json | \
    python3 -c "import json, sys; print('\n'.join([x['name'] for x in json.load(sys.stdin)]))" | \
    xargs -n1 pip install -U

echo "Update completed!"
EOF

chmod +x ~/scripts/update-system.sh
```

### 5. Compactar disco virtual do WSL2

```powershell
# PowerShell como Admin
# Desligar WSL2
wsl --shutdown

# Compactar disco virtual
Optimize-VHD -Path C:\Users\Minoru\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_*\LocalState\ext4.vhdx -Mode Full

# Ou usar diskpart
diskpart
# select vdisk file="C:\Users\Minoru\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_*\LocalState\ext4.vhdx"
# compact vdisk
# exit
```

---

## Troubleshooting

### Problema: WSL2 não inicia

```powershell
# Verificar status
wsl --status

# Reiniciar WSL
wsl --shutdown
wsl

# Se persistir, verificar logs
wsl --verbose
```

### Problema: Docker não funciona no WSL2

```bash
# Verificar se Docker daemon está rodando
docker info

# Reiniciar Docker Desktop no Windows
# Verificar integração WSL2 nas configurações
```

### Problema: Conexão Oracle falha

```bash
# Verificar variáveis de ambiente
echo $ORACLE_HOME
echo $LD_LIBRARY_PATH

# Verificar conectividade de rede
ping oracle-host
telnet oracle-host 1521

# Testar com tnsping
tnsping "your-tns-entry"
```

### Problema: Performance lenta

```bash
# Verificar uso de recursos
htop

# Verificar uso de disco
df -h
du -sh ~/projects/*

# Verificar se está usando filesystem WSL2 (rápido) ou /mnt/c (lento)
pwd
# Projetos devem estar em /home/minoru, não em /mnt/c
```

### Problema: Localhost não funciona entre Windows e WSL2

```bash
# Verificar se localhost forwarding está habilitado no .wslconfig
cat /mnt/c/Users/Minoru/.wslconfig

# Usar IP do WSL2 se necessário
ip addr show eth0 | grep inet
```

### Problema: Git credential helper não funciona

```bash
# Reconfigurar
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"

# Ou usar SSH keys
ssh-keygen -t ed25519 -C "seu-email@empresa.com"
cat ~/.ssh/id_ed25519.pub
# Adicionar a chave no Azure DevOps
```

---

## Comandos Úteis de Referência Rápida

### WSL2

```powershell
# Windows PowerShell
wsl --list --verbose              # Listar distribuições
wsl --shutdown                    # Desligar todas as instâncias
wsl --terminate Ubuntu-22.04      # Desligar distro específica
wsl --export Ubuntu-22.04 backup.tar   # Exportar
wsl --import Nome Path backup.tar      # Importar
wsl --unregister Ubuntu-22.04     # Remover distribuição
```

### Docker

```bash
# WSL2 bash
docker ps                         # Containers rodando
docker ps -a                      # Todos os containers
docker images                     # Listar imagens
docker system prune -a            # Limpar tudo
docker-compose up -d              # Subir stack em background
docker-compose logs -f            # Ver logs
```

### Kubernetes

```bash
# WSL2 bash
kubectl get pods                  # Listar pods
kubectl get services              # Listar services
kubectl logs pod-name             # Ver logs
kubectl describe pod pod-name     # Detalhes do pod
kubectl apply -f manifest.yaml    # Aplicar manifest
kubectl delete -f manifest.yaml   # Remover recursos
minikube dashboard                # Abrir dashboard local
```

### Python

```bash
# WSL2 bash
source venv/bin/activate          # Ativar ambiente virtual
deactivate                        # Desativar ambiente virtual
pip freeze > requirements.txt     # Salvar dependências
pip install -r requirements.txt   # Instalar dependências
python -m pytest                  # Executar testes
python -m black .                 # Formatar código
```

---

## Próximos Passos

1. ✅ Instalar WSL2 e Ubuntu 22.04
2. ✅ Configurar Python e FastAPI
3. ✅ Instalar Docker e Kubernetes
4. ✅ Configurar Oracle Client
5. ✅ Integrar com VS Code
6. ⏭️ Clonar repositório do fraud-detection do Azure DevOps
7. ⏭️ Configurar CI/CD pipeline para deploy no OKE
8. ⏭️ Criar runbook de desenvolvimento no Obsidian
9. ⏭️ Documentar processo no ADR do IntelligencePortal

---

## Recursos Adicionais

- **Documentação oficial WSL2:** https://learn.microsoft.com/en-us/windows/wsl/
- **FastAPI Docs:** https://fastapi.tiangolo.com/
- **Docker WSL2 Backend:** https://docs.docker.com/desktop/wsl/
- **Oracle Instant Client:** https://www.oracle.com/database/technologies/instant-client.html
- **VS Code Remote Development:** https://code.visualstudio.com/docs/remote/wsl

---

**Versão do documento:** 1.0  
**Última atualização:** 26/02/2025
