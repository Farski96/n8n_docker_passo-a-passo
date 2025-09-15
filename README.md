# n8n_docker_passo-a-passo

📘 Estrutura recomendada do repo
  ├── README.md
  ├── docker-compose.yml
  ├── .env.example
  └── .gitignore


O .gitignore deve conter .env e postgres-data/ e n8n/.

Apenas o .env.example fica versionado.

📑 README.md
# 🚀 n8n — Instalação Local e com Docker

Este repositório traz um guia profissional e arquivos prontos para rodar o **n8n**, seja para testes locais ou produção com **Docker + Postgres**.

---

## 📂 Estrutura
- `docker-compose.yml` → Orquestra n8n + Postgres.  
- `.env.example` → Exemplo de variáveis de ambiente (copie para `.env`).  
- `postgres-data/` → Volume do banco (ignorado pelo Git).  
- `n8n/` → Configurações e dados do n8n (ignorado pelo Git).  

---

## 🛠️ Opções de instalação

### 1️⃣ Local (npm) — rápido para testes
```bash
# Instale Node.js (LTS recomendado)
npm install -g n8n

# Rode o n8n
n8n
# ou
n8n start


Acesse em: http://localhost:5678

2️⃣ Docker — rápido
docker run -it --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=sua-senha \
  n8nio/n8n

3️⃣ Docker Compose — recomendado (produção leve)

Copie o arquivo .env.example para .env:

cp .env.example .env


Edite os valores (senhas, chaves, host).

Suba os serviços:

docker compose up -d


Acesse em: http://localhost:5678

⚙️ Atualização
docker compose pull
docker compose up -d --force-recreate

🔒 Segurança & Boas práticas

Sempre defina N8N_ENCRYPTION_KEY em produção.

Não exponha o n8n sem proxy HTTPS. Use Nginx/Traefik/Caddy.

Nunca versione o arquivo .env.

Faça backup do banco e dos exports (n8n export:workflow).

📊 Backup rápido (CLI)
# Workflows
n8n export:workflow --all --output=backups/workflows.json

# Credenciais (descriptografadas)
n8n export:credentials --all --decrypted --output=backups/credentials.json

✅ Checklist Produção

 .env configurado com senha forte e chave de encriptação

 Banco Postgres com backup automatizado

 Proxy reverso com HTTPS configurado

 Backup regular dos workflows/credenciais

⭐ Se este guia te ajudou, deixe uma estrela no repositório!


---

# 🐳 docker-compose.yml

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - ./postgres-data:/var/lib/postgresql/data

  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: ${N8N_BASIC_AUTH_USER}
      N8N_BASIC_AUTH_PASSWORD: ${N8N_BASIC_AUTH_PASSWORD}
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      NODE_ENV: production
    volumes:
      - ./n8n:/home/node/.n8n
    depends_on:
      - postgres

📄 .env.example
# Banco de Dados
POSTGRES_USER=n8n
POSTGRES_PASSWORD=troque-senha-postgres
POSTGRES_DB=n8n

# Autenticação da UI
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=troque-senha-ui

# Criptografia de credenciais
N8N_ENCRYPTION_KEY=chave-muito-longa-e-secreta-gera-uma-aleatoria

# Host/Protocolo (para webhooks em produção)
WEBHOOK_URL=https://seu-dominio.com/
N8N_PROTOCOL=https
N8N_HOST=seu-dominio.com
N8N_PORT=443
