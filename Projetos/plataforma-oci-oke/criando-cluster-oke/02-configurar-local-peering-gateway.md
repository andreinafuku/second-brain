---
title: "Configurar Local Peering Gateway"
date: 2026-04-23
tags:
  - projeto
  - oci
  - oke
  - rede
  - runbook
  - status/rascunho
area: plataforma
---

# Configurar Local Peering Gateway (LPG)

## Introdução

O **Local Peering Gateway (LPG)** é um mecanismo OCI que permite conectar duas Virtual Cloud Networks (VCNs) na mesma região, permitindo comunicação privada entre recursos das duas redes.

### Por Que Usar LPG?

- ✅ Comunicação privada entre VCNs (sem passar por internet pública)
- ✅ Baixa latência (mesma região)
- ✅ Sem custo de dados transferidos
- ✅ Fácil configuração em OCI

### Problema Específico do Projeto

- **VCN OKE (nova):** 10.2.0.0/16 - Cluster Kubernetes
- **VCN xrt-interno:** 10.0.0.0/16 - Banco de dados Oracle
- **Objetivo:** Permitir que pods no OKE acessem o banco via LPG

## Pré-Requisitos

- ✅ Cluster OKE criado com VCN 10.2.0.0/16
- ✅ VCN xrt-interno com CIDR 10.0.0.0/16 existente
- ✅ CIDRs não conflitam (não se sobrepõem)
- ✅ Acesso ao OCI Console com permissões de:
  - `vcn:ManageLocalPeeringGateways`
  - `core:ManageRouteTables`
  - `core:ManageSecurityLists`
- ✅ Conhecimento dos IP ranges das subnets (especialmente banco de dados)

## Parte 1: Criar Local Peering Gateways

### Passo 1.1: Criar LPG na VCN OKE

1. Navegue até **Virtual Cloud Networks** → `oke-treasury-vcn`
2. Na seção **Resources** (esquerda), clique em **Local Peering Gateways**
3. Clique em **Create Local Peering Gateway**
4. Preencha:
   - **Name:** `lpg-oke-to-xrt`
   - **Description:** `LPG connecting OKE cluster to xrt-interno VCN for database access`
   - **Peer VCN in the same region:** `xrt-interno`
   - **Peer Local Peering Gateway:** (será criado ou associado)

5. Clique em **Create Local Peering Gateway**

> ⏱️ O LPG será criado em estado "NEW" (aguardando aceitação pela VCN peer).

### Passo 1.2: Criar LPG na VCN xrt-interno

1. Navegue até **Virtual Cloud Networks** → `xrt-interno`
2. Na seção **Resources**, clique em **Local Peering Gateways**
3. Clique em **Create Local Peering Gateway**
4. Preencha:
   - **Name:** `lpg-xrt-to-oke`
   - **Description:** `LPG connecting xrt-interno VCN to OKE cluster for database connectivity`
   - **Peer VCN in the same region:** `oke-treasury-vcn`
   - **Peer Local Peering Gateway:** `lpg-oke-to-xrt` (o que criamos anteriormente)

5. Clique em **Create Local Peering Gateway**

> ✅ Após este passo, os dois LPGs estarão em estado "PEERED" (conectados).

### Passo 1.3: Validar Peering Status

1. Volte para **Virtual Cloud Networks** → `oke-treasury-vcn` → **Local Peering Gateways**
2. Verifique que `lpg-oke-to-xrt` está em status **"PEERED"**
3. Verifique os detalhes:
   - **Peer VCN:** `xrt-interno` (OCID completo)
   - **Peer Local Peering Gateway:** `lpg-xrt-to-oke`

Repita o mesmo para a VCN xrt-interno.

## Parte 2: Configurar Route Tables

As route tables determinam como o tráfego é roteado. Você precisa adicionar rotas que indiquem para os LPGs.

### Passo 2.1: Adicionar Rota na VCN OKE

Na VCN OKE, adicione uma rota que direcione tráfego destinado ao banco de dados (164.152.38.0/24) através do LPG.

1. Navegue até **Virtual Cloud Networks** → `oke-treasury-vcn` → **Route Tables**
2. Clique na route table `oke-public-route` (ou a que está associada à subnet dos workers)
3. Clique em **Add Route Rule**
4. Preencha:
   - **Destination CIDR Block:** `164.152.38.0/24` (subnet do banco de dados)
   - **Target Type:** `Local Peering Gateway`
   - **Target Local Peering Gateway:** `lpg-oke-to-xrt`
   - **Description:** `Route to xrt-interno database subnet via LPG`

5. Clique em **Add Route Rule**

> ✅ Tráfego da VCN OKE destinado a 164.152.38.0/24 será roteado via LPG.

### Passo 2.2: Adicionar Rota na VCN xrt-interno

Agora faça o mesmo na VCN xrt-interno para aceitar tráfego vindo da VCN OKE.

1. Navegue até **Virtual Cloud Networks** → `xrt-interno` → **Route Tables**
2. Clique na route table associada à subnet do banco de dados (ex: `Default Route Table for xrt-interno`)
3. Clique em **Add Route Rule**
4. Preencha:
   - **Destination CIDR Block:** `10.2.0.0/16` (toda a VCN OKE)
   - **Target Type:** `Local Peering Gateway`
   - **Target Local Peering Gateway:** `lpg-xrt-to-oke`
   - **Description:** `Route to OKE cluster subnet via LPG`

5. Clique em **Add Route Rule**

> ✅ Tráfego destinado à VCN OKE será roteado via LPG na direção oposta.

## Parte 3: Configurar Security Lists

Security lists funcionam como firewalls em nível de subnet. Você deve permitir especificamente o tráfego entre as subnets.

### Passo 3.1: Configurar Security List da Subnet Workers (OKE)

Adicione regra de egress para permitir tráfego ao banco de dados.

1. Navegue até **Virtual Cloud Networks** → `oke-treasury-vcn` → **Security Lists**
2. Clique em `oke-default-seclist`
3. Na seção **Egress Rules**, clique em **Add Egress Rule**
4. Preencha:
   - **Stateless:** Desmarque (para manter estado da conexão)
   - **Direction:** Egress
   - **Destination Type:** CIDR
   - **Destination CIDR:** `164.152.38.0/24` (subnet do banco)
   - **IP Protocol:** TCP
   - **Destination Port Range:** `1521` (porta Oracle DB)
   - **Description:** `Oracle Database connectivity via LPG`

5. Clique em **Add Egress Rule**

> ✅ Pods na subnet workers (10.2.10.0/24) agora podem conectar ao banco na porta 1521.

### Passo 3.2: Configurar Security List da Subnet Database (xrt-interno)

Adicione regra de ingress para aceitar tráfego do OKE.

1. Navegue até **Virtual Cloud Networks** → `xrt-interno` → **Security Lists**
2. Clique na security list associada à subnet do banco (ex: `Default Security List for xrt-interno`)
3. Na seção **Ingress Rules**, clique em **Add Ingress Rule**
4. Preencha:
   - **Stateless:** Desmarque
   - **Direction:** Ingress
   - **Source Type:** CIDR
   - **Source CIDR:** `10.2.10.0/24` (subnet workers OKE)
   - **IP Protocol:** TCP
   - **Destination Port Range:** `1521` (porta Oracle DB)
   - **Description:** `Accept OKE cluster database requests via LPG`

5. Clique em **Add Ingress Rule**

> ✅ A subnet do banco agora aceita conexões da subnet workers do OKE.

## Parte 4: Configurar Network Security Groups (NSGs) - Opcional

Se a VCN xrt-interno usa NSGs além de security lists, você pode ter regras mais granulares.

### Passo 4.1: Verificar NSGs Existentes

1. Navegue até **Virtual Cloud Networks** → `xrt-interno` → **Network Security Groups**
2. Identifique quais NSGs estão associados às instâncias do banco de dados
3. Anote os nomes (ex: `xrt-database-nsg`, `oracle-db-nsg`, etc.)

### Passo 4.2: Adicionar Regra de Ingress no NSG

1. Clique no NSG do banco de dados
2. Clique em **Add Ingress Rule**
3. Preencha:
   - **Stateless:** Desmarque
   - **Source:** `CIDR`
   - **Source CIDR:** `10.2.10.0/24` (subnet workers OKE)
   - **Protocol:** TCP
   - **Destination Port Range:** `1521`
   - **Description:** `OKE cluster to Oracle DB`

4. Clique em **Add Ingress Rule**

> 💡 Se NSGs não existem, apenas security lists são suficientes (passo 3.2).

## Parte 5: Testes de Conectividade

### Passo 5.1: Criar Pod de Teste

```bash
# Criar namespace para testes
kubectl create namespace test

# Criar pod de teste (busybox com netcat)
kubectl run -n test debug --image=busybox --restart=Never -it -- /bin/sh
```

### Passo 5.2: Testar Conectividade TCP

Dentro do pod:

```bash
# Teste 1: Verificar se consegue resolver DNS (opcional)
nslookup kubernetes.default
nslookup 8.8.8.8

# Teste 2: Testar conexão TCP à porta 1521 do banco
nc -zv 164.152.38.179 1521

# Resultado esperado:
# Connection to 164.152.38.179 1521 port [tcp/oraclelistener] succeeded!

# Teste 3: Testar com timeout (se nc não responder rápido)
timeout 5 bash -c 'cat > /dev/null < /dev/tcp/164.152.38.179/1521'
echo $?  # 0 = sucesso, outro = falha
```

### Passo 5.3: Teste com SQLPlus (Mais Realista)

Se o SQLPlus está disponível:

```bash
# Dentro do pod ou na máquina local
sqlplus -v  # Verificar se está instalado

# Tentar conexão
sqlplus <USER>/<PASSWORD>@164.152.38.179:1521/ORCL

# Ou via JDBC (aplicação Java)
# Connection URL: jdbc:oracle:thin:@164.152.38.179:1521:ORCL
```

### Passo 5.4: Teste de Latência

```bash
# Medir latência com ping (se ICMP está permitido)
ping -c 4 164.152.38.179

# Ou com traceroute para ver o caminho
traceroute 164.152.38.179
```

### Cleanup após testes

```bash
# Remover pod de teste
kubectl delete pod -n test debug

# Remover namespace
kubectl delete namespace test
```

## Validação Completa

### Checklist de Validação LPG

- [ ] LPG `lpg-oke-to-xrt` existe e está em status "PEERED"
- [ ] LPG `lpg-xrt-to-oke` existe e está em status "PEERED"
- [ ] Route table de OKE tem rota para 164.152.38.0/24 → `lpg-oke-to-xrt`
- [ ] Route table de xrt-interno tem rota para 10.2.0.0/16 → `lpg-xrt-to-oke`
- [ ] Security list OKE permite egress TCP/1521 para 164.152.38.0/24
- [ ] Security list xrt-interno permite ingress TCP/1521 de 10.2.10.0/24
- [ ] Pod de teste consegue conectar em 164.152.38.179:1521
- [ ] Aplicação no OKE consegue conectar ao banco

## Troubleshooting

### Problema: LPG status é "REJECTED"

**Causa:** VCN peer não aceitou a requisição de peering.

**Solução:**
1. Verifique se os LPGs foram criados em ambas as VCNs
2. Navegue até a VCN peer e procure por LPG "pendente"
3. Clique em "Accept" no LPG pendente

### Problema: LPG status é "INVALID"

**Causa:** CIDRs das VCNs se sobrepõem.

**Solução:**
- Verifique os CIDRs: OKE deve ser 10.2.0.0/16, xrt-interno 10.0.0.0/16
- Se houver conflito, delete o cluster OKE antigo (com CIDR 10.0.0.0/16)

### Problema: Pod não consegue conectar ao banco

**Checklist:**
1. Verifique route tables:
   ```bash
   # Obter informações da subnet workers
   oci network subnet get --subnet-id <SUBNET-OCID>
   ```

2. Verifique security lists:
   ```bash
   # Listar rules
   oci network security-list list --vcn-id <VCN-OCID>
   ```

3. Teste conectividade simples:
   ```bash
   kubectl run -it --rm debug --image=busybox -- nc -zv 164.152.38.179 1521
   ```

4. Verifique logs do pod:
   ```bash
   kubectl logs -n ts-dev <POD-NAME>
   ```

### Problema: Tráfego é lento ou intermitente

**Causa:** LPG pode estar em estado "SUSPENDED" ou com problemas.

**Solução:**
1. Verifique status no OCI Console:
   ```bash
   oci ce local-peering-gateway list --vcn-id <VCN-OCID>
   ```

2. Se necessário, delete e recrie o LPG:
   ```bash
   # OCI Console: Delete LPG → Recrie seguindo o passo 1
   ```

### Problema: Aplicação conecta mas muito lento

**Possíveis causas:**
- Pool de conexões mal configurado
- Aplicação fazendo múltiplas conexões simultâneas
- Problema de DNS resolution

**Solução:**
1. Verificar pool de conexões na aplicação
2. Verificar se DNS resolve corretamente:
   ```bash
   kubectl run -it --rm debug --image=busybox -- nslookup 164.152.38.179
   ```

3. Usar ferramentas de diagnóstico:
   ```bash
   kubectl exec -it <POD> -- /bin/sh
   tcpdump -i any -n 'tcp port 1521' # Capturar tráfego TCP
   ```

## Próximos Passos

✅ Local Peering Gateway está configurado e testado

**Próximos passos recomendados:**
1. **[`03-configurar-ingress-nginx.md`](./03-configurar-ingress-nginx.md)** - Instalar NGINX Ingress Controller
2. **[`04-configurar-cert-manager.md`](./04-configurar-cert-manager.md)** - Configurar certificados TLS
3. **[`05-configurar-monitoring.md`](./05-configurar-monitoring.md)** - Configurar monitoring
4. **[`06-migrar-aplicacao.md`](./06-migrar-aplicacao.md)** - Migrar aplicação DeepTreasury

## Referências

- [Oracle LPG Documentation](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/localVCNpeering.htm)
- [OCI VCN Best Practices](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/bestpracticesnetwork.htm)
- [OCI CLI LPG Commands](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/network/local-peering-gateway.html)

---

**Última atualização:** Fevereiro 2026
**Status:** Documento completo
