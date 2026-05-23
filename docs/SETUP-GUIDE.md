# 🚀 Guia de Setup - SCA Auto Remediation

## Pré-requisitos

- ✅ Git instalado
- ✅ PHP 8.3+
- ✅ Composer instalado
- ✅ Python 3.11+
- ✅ Conta GitHub
- ✅ Conta Supabase (gratuita)

---

## Passo 1: Preparar Supabase (PostgreSQL Gerenciado)

### 1.1 Criar Projeto

```bash
# Acesse: https://app.supabase.com
# Clique: New Project
# Dados:
#   Name: open-innovation-priscila
#   Database Password: use-strong-password
#   Region: sa-east-1 (São Paulo) ou mais próximo
#   Clique: Create new project
```

### 1.2 Copiar Credenciais

Após criar o projeto:

```bash
# Em Supabase → Settings → Database
POSTGRES_HOST = db.xxxxx.supabase.co
POSTGRES_USER = postgres
POSTGRES_PASSWORD = xxxxx

# Em Supabase → Settings → API
SUPABASE_URL = https://xxxxx.supabase.co
SUPABASE_ANON_KEY = eyJhbG...
SUPABASE_SERVICE_KEY = eyJhbG...
```

### 1.3 Inicializar Banco de Dados

```bash
# Opção 1: Via psql (local)
psql -h db.xxxxx.supabase.co \
  -U postgres \
  -d postgres \
  -c "CREATE DATABASE open_innovation;"

psql -h db.xxxxx.supabase.co \
  -U postgres \
  -d open_innovation \
  -f scripts/init-db.sql

# Opção 2: Via Supabase Console (Web)
# → SQL Editor → New Query → Cole conteúdo init-db.sql → Run
```

---

## Passo 2: Configurar GitHub Secrets

### 2.1 Navegue até o Repositório

```bash
# GitHub → seu repo → Settings → Secrets and variables → Actions
```

### 2.2 Adicionar Secrets

Clique **New repository secret** e adicione:

| Nome | Valor |
|------|-------|
| `SUPABASE_URL` | `https://xxxxx.supabase.co` |
| `SUPABASE_ANON_KEY` | `eyJhbG...` |
| `SUPABASE_SERVICE_KEY` | `eyJhbG...` |
| `SLACK_WEBHOOK` | `https://hooks.slack.com/...` (opcional) |

### 2.3 Testar Conectividade

```bash
# Local
export POSTGRES_HOST=db.xxxxx.supabase.co
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=xxxxx
export POSTGRES_DB=open_innovation

python3 -c "import psycopg2; psycopg2.connect(host='$POSTGRES_HOST', user='$POSTGRES_USER', password='$POSTGRES_PASSWORD', database='$POSTGRES_DB')" && echo "✅ Conexão OK"
```

---

## Passo 3: Preparar Repositório Local

### 3.1 Clone o Repositório

```bash
git clone https://github.com/fcborges/open-innovation-priscila.git
cd open-innovation-priscila
```

### 3.2 Checkout da Branch Develop

```bash
git checkout develop
git pull origin develop
```

### 3.3 Instalar Dependências PHP

```bash
composer install
```

### 3.4 Instalar Dependências Python

```bash
pip install -r requirements.txt
# Ou manualmente:
pip install requests pyyaml psycopg2-binary python-dotenv
```

---

## Passo 4: Criar Arquivo .env (Local)

### 4.1 Criar `.env.local`

```bash
cat > .env.local << 'EOF'
# PostgreSQL
POSTGRES_HOST=db.xxxxx.supabase.co
POSTGRES_USER=postgres
POSTGRES_PASSWORD=xxxxx
POSTGRES_DB=open_innovation

# Supabase
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=eyJhbG...
SUPABASE_SERVICE_KEY=eyJhbG...

# OSV.dev (público, sem credenciais)
OSV_API_URL=https://api.osv.dev/v1/query

# Modo debug
DEBUG=true
EOF
```

### 4.2 Não Commitar `.env.local`

```bash
echo ".env.local" >> .gitignore
git add .gitignore
git commit -m "chore: Ignore .env.local"
```

---

## Passo 5: Testar Localmente

### 5.1 Carregar Políticas no Banco

```bash
source .env.local
python3 scripts/load_policies.py
```

**Resultado esperado:**
```
2026-05-23 12:00:00 - INFO - 🔄 Sincronizando políticas de homologação...
2026-05-23 12:00:01 - INFO - 📦 Carregadas 30 políticas do YAML
2026-05-23 12:00:02 - INFO - ✅ 30 políticas sincronizadas ao banco
```

### 5.2 Executar SCA Manual

```bash
source .env.local
python3 scripts/sca_remediation.py
```

**Resultado esperado:**
```
2026-05-23 12:00:00 - INFO - 🚀 Iniciando SCA Remediation Engine
2026-05-23 12:00:01 - INFO - ✅ Conectado ao PostgreSQL
2026-05-23 12:00:02 - INFO - 📦 Lido 45 packages do composer.lock
2026-05-23 12:00:35 - INFO - 📄 Relatórios gerados em .reports
2026-05-23 12:00:36 - INFO - ✅ SCA Remediation concluído com sucesso
```

### 5.3 Inspecionar Relatórios

```bash
cat .reports/sca_report.json | jq .
```

---

## Passo 6: Fazer Push e Ativar Pipeline

### 6.1 Fazer Commit

```bash
git add -A
git commit -m "feat: Implementa SCA Auto Remediation (Opção 2 - PostgreSQL + Supabase)"
git push origin develop
```

### 6.2 Monitorar Execução

```bash
# GitHub → Actions → SCA Auto Remediation
# Aguarde ~2-5 minutos
```

**Quando terminar, você verá:**
- ✅ Build bem-sucedido OU
- ❌ Erro (com logs detalhados)

### 6.3 Inspecionar Logs

Clique no workflow → job `sca-remediation` → veja outputs

---

## Passo 7: Criar Merge Request para Main

### 7.1 Criar PR

```bash
git push origin develop
# GitHub → Create Pull Request
# Base: main ← Compare: develop
```

### 7.2 Validar Check da Pipeline

Verifique se a SCA passou:
- ✅ SCA Auto Remediation
- ✅ Commit automático (se houver atualizações)

### 7.3 Merge

```bash
# GitHub UI → Merge Pull Request
# Ou terminal:
git checkout main
git pull origin main
git merge develop
git push origin main
```

---

## Passo 8: Validar Banco de Dados

### 8.1 Consultar Políticas

```bash
# Via psql
psql -h db.xxxxx.supabase.co \
  -U postgres \
  -d open_innovation \
  -c "SELECT package_name, approved_version FROM approved_packages LIMIT 10;"
```

**Resultado esperado:**
```
      package_name      | approved_version
------------------------+------------------
laravel/framework       | 11.20.0
guzzlehttp/guzzle       | 7.9.2
symfony/http-foundation | 6.4.8
...
```

### 8.2 Ver Vulnerabilidades Detectadas

```bash
psql -h db.xxxxx.supabase.co \
  -U postgres \
  -d open_innovation \
  -c "SELECT * FROM security.active_vulnerabilities LIMIT 10;"
```

### 8.3 Ver Remediações Realizadas

```bash
psql -h db.xxxxx.supabase.co \
  -U postgres \
  -d open_innovation \
  -c "SELECT * FROM security.recent_remediations;"
```

---

## Passo 9: Agendar Execução Diária

A pipeline já está configurada para rodar:
- ✅ Todo dia às 08:00 UTC
- ✅ Em todo push para main/develop
- ✅ Em todo PR para main/develop

Se quiser ajustar:

```yaml
# .github/workflows/sca-remediation.yml
schedule:
  - cron: '0 8 * * *'  # Ajuste conforme necessário
```

---

## Passo 10: Configurar Notificações (Opcional)

### 10.1 Slack Integration

```bash
# Slack → Create app → Incoming Webhooks
# Copie o webhook
```

### 10.2 Adicionar Secret

```bash
# GitHub → Settings → Secrets → SLACK_WEBHOOK
# Valor: https://hooks.slack.com/services/...
```

### 10.3 Receber Notificações

Agora você receberá:
- ✅ SCA Completo
- 🚨 Vulnerabilidades encontradas
- ❌ Erros na pipeline

---

## ✅ Checklist Final

- [ ] Supabase project criado
- [ ] Banco inicializado (init-db.sql)
- [ ] GitHub Secrets configurados
- [ ] `.env.local` criado (não commitado)
- [ ] Dependências PHP instaladas
- [ ] Dependências Python instaladas
- [ ] SCA rodar localmente com sucesso
- [ ] Push para develop feito
- [ ] Pipeline executou com sucesso
- [ ] PR criado e merged para main
- [ ] Banco contém políticas (aprovadas)
- [ ] Banco contém vulnerabilidades (se detectadas)

---

## 🔧 Troubleshooting

### Erro: "Could not connect to PostgreSQL"

```bash
# Verifique credenciais
echo $POSTGRES_HOST
echo $POSTGRES_USER

# Teste conexão
psql -h $POSTGRES_HOST -U $POSTGRES_USER -d postgres -c "SELECT 1;"
```

### Erro: "No such file or directory: init-db.sql"

```bash
# Certifique-se que está no diretório correto
pwd  # deve ser: /path/to/open-innovation-priscila

# Execute:
python3 scripts/load_policies.py
```

### Pipeline não executa

```bash
# Verifique se arquivo existe:
cat .github/workflows/sca-remediation.yml

# Force uma execução manual:
# GitHub → Actions → SCA Auto Remediation → Run workflow
```

### "Access denied" no GitHub

```bash
# Verifique SSH/HTTPS
git remote -v

# Se HTTPS:
git remote set-url origin git@github.com:fcborges/open-innovation-priscila.git

# Ou configure PAT:
git config --global credential.helper store
git pull  # Pedirá credenciais
```

---

## 📞 Suporte

Dúvidas? Verifique:

1. **Logs locais**: `.logs/sca_remediation.log`
2. **Logs da pipeline**: GitHub Actions
3. **Banco de dados**: Supabase Console → SQL Editor
4. **Documentação**: `docs/SCA-ARCHITECTURE.md`

---

**Pronto!** 🎉

Sua pipeline SCA está operacional e automatizará a remediação de vulnerabilidades PHP em tempo real.
