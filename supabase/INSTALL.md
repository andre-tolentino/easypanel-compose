# Supabase Self-Hosted com S3 MinIO no Easypanel

## Pré-requisitos

- Servidor Linux com Easypanel instalado
- Conta no GitHub
- Domínio apontando para o servidor

---

## Passo 1: Forkar o Repositório

1. Acesse `https://github.com/easypanel-io/compose`
2. Clique em **Fork**
3. Selecione sua conta GitHub
4. Aguarde o fork ser criado

---

## Passo 2: Subir as Alterações para o GitHub

No seu computador, abra o terminal e execute:

```bash
# Clonar seu fork
git clone https://github.com/SEUUsuario/compose.git
cd compose

# Copiar os arquivos atualizados
# (substitua os arquivos na pasta supabase/code/)

# Fazer commit e push
git add .
git commit -m "Add MinIO S3 storage and expose Supavisor ports"
git push
```

---

## Passo 3: Configurar no Easypanel

### 3.1 Criar o Projeto

1. No Easypanel, clique em **New Project**
2. Nome: `supabase`
3. Clique em **Create**

### 3.2 Configurar Variáveis de Ambiente

Vá em **Settings > Environment** e adicione todas as variáveis do `.env`:

```
POSTGRES_PASSWORD=sua_senha_forte_aqui
JWT_SECRET=sua_chave_jwt_aqui
ANON_KEY=seu_anon_key_aqui
SERVICE_ROLE_KEY=seu_service_role_key_aqui
DASHBOARD_USERNAME=supabase
DASHBOARD_PASSWORD=sua_senha_do_dashboard_aqui
SECRET_KEY_BASE=sua_secret_key_base_aqui
VAULT_ENC_KEY=sua_vault_enc_key_aqui
POSTGRES_HOST=db
POSTGRES_DB=postgres
POSTGRES_PORT=5432
POOLER_PROXY_PORT_TRANSACTION=6543
POOLER_DEFAULT_POOL_SIZE=20
POOLER_MAX_CLIENT_CONN=100
POOLER_TENANT_ID=your-tenant-id
SUPABASE_PUBLIC_URL=https://seu_dominio.com
API_EXTERNAL_URL=https://seu_dominio.com
SITE_URL=https://seu_dominio.com
MINIO_ROOT_USER=supa-storage
MINIO_ROOT_PASSWORD=secret1234
GLOBAL_S3_BUCKET=supabase
REGION=us-east-1
```

### 3.3 Criar o Compose Service

1. Clique em **New Service > Compose**
2. Nome: `supabase`
3. **Source**: Git
4. **Repository**: `https://github.com/SEUUsuario/compose.git`
5. **Branch**: `main`
6. **Root Path**: `/supabase/code`
7. **Compose File**: `docker-compose.yml`
8. Clique em **Create**

### 3.4 Configurar o Domínio

1. Vá no serviço **`supabase-kong`**
2. **Domains & Proxy > Add Domain**
3. Domínio: `seu_dominio.com`
4. **Proxy Port**: `8000`
5. Marque como **Primary Domain**
6. **Save**

---

## Passo 4: Deploy

1. Clique em **Deploy**
2. Aguarde todos os serviços ficarem healthy
3. Acesse `https://seu_dominio.com`
4. Login: `supabase` / `sua_senha_do_dashboard`

---

## Portas Expostas

| Serviço | Porta | Protocolo | Uso |
|---|---|---|---|
| Kong | 8000 | HTTP | API Gateway |
| Kong | 8443 | HTTPS | API Gateway (TLS) |
| Supavisor | 5432 | TCP | Conexão Postgres (session) |
| Supavisor | 6543 | TCP | Conexão Postgres (transaction) |
| MinIO | 9000 | TCP | S3 API |
| MinIO | 9001 | TCP | MinIO Console |

---

## Connection Strings

### Postgres (via Supavisor)

**Session mode (porta 5432):**
```
postgresql://postgres.your-tenant-id:sua_senha@IP_SERVIDOR:5432/postgres
```

**Transaction mode (porta 6543):**
```
postgresql://postgres.your-tenant-id:sua_senha@IP_SERVIDOR:6543/postgres
```

### MinIO Console

```
http://IP_SERVIDOR:9001
```

Login: `supa-storage` / `secret1234`

### MinIO S3 API

```
http://IP_SERVIDOR:9000
```

---

## Verificação

### Testar portas externamente

```powershell
# PowerShell
Test-NetConnection -ComputerName IP_SERVIDOR -Port 5432
Test-NetConnection -ComputerName IP_SERVIDOR -Port 6543
Test-NetConnection -ComputerName IP_SERVIDOR -Port 9001
```

### Testar MinIO

Acesse `http://IP_SERVIDOR:9001` e faça login com as credenciais do MinIO.

### Testar Supabase Studio

Acesse `https://seu_dominio.com` e faça login com as credenciais do Studio.

---

## Solução de Problemas

| Problema | Solução |
|---|---|
| Analytics não inicia | Verificar se o schema `_supabase` foi criado |
| Storage não funciona | Verificar se MinIO está rodando e o bucket foi criado |
| Portas não expostas | Verificar firewall do servidor |
| JWT verification failed | Regenerar tokens JWT com script correto |
| Studio sem login | Verificar se domínio aponta para Kong (porta 8000) |

---

## Arquivos Modificados

| Arquivo | Alteração |
|---|---|
| `docker-compose.yml` | Adicionado MinIO, Supavisor ports, Storage S3 |
| `.env` | Adicionadas variáveis MinIO e S3 |
