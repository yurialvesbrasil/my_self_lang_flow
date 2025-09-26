## LangFlow

![Run LangChain Locally](docs/img/lang_flow.png)

Para acessar a aplicação: http://localhost:7860/

### Visão geral

Este repositório orquestra o LangFlow via Docker Compose com serviços de Postgres e Redis. O nome do projeto no Docker é LangFlow (container principal `langflow`).

### Tecnologias e bibliotecas

- **Docker / Docker Compose**: containerização e orquestração dos serviços
- **LangFlow**: `langflowai/langflow:latest` executando a aplicação
- **Postgres 15**: banco de dados relacional
- **Redis 7**: cache/filas e suporte a bundles

### Padrões adotados

- **12-factor config**: variáveis no arquivo `.env`
- **Infra como código**: definição de serviços em `docker-compose.yml`
- **Persistência**: volumes nomeados para dados (`postgres_data`, `redis_data`, `langflow_data`) e mapeamento de flows (`./flows -> /app/flows`)

### Setup rápido

1. Pré-requisitos: Docker e Docker Compose instalados
2. Crie um arquivo `.env` na raiz com as variáveis mínimas:

```env
# LangFlow
LANGFLOW_PORT=7860
LANGFLOW_SECRET_KEY=change-me
LANGFLOW_DATABASE_URL=postgresql+psycopg://postgres:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
LANGFLOW_REDIS_URL=redis://default:${REDIS_PASSWORD}@redis:6379/0
LANGFLOW_AUTO_SAVING=true
LANGFLOW_AUTO_SAVING_INTERVAL=1000
LANGFLOW_LOAD_FLOWS_PATH=/app/flows

# Postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=langflow

# Redis
REDIS_PASSWORD=redis
```

3. Opcional: coloque seus flows em `./flows` (serão montados em `/app/flows`)
4. Suba os serviços:

```bash
docker compose up -d
```

### Serviços (docker-compose.yml)

- **langflow**
  - image: `langflowai/langflow:latest`
  - container_name: `langflow`
  - porta: `7860:7860`
  - env: `LANGFLOW_PORT`, `LANGFLOW_DATABASE_URL`, `LANGFLOW_REDIS_URL`, `LANGFLOW_SECRET_KEY`, `LANGFLOW_AUTO_SAVING`, `LANGFLOW_AUTO_SAVING_INTERVAL`, `LANGFLOW_LOAD_FLOWS_PATH`
  - volumes: `./flows:/app/flows`, `langflow_data:/langflow-data`
  - depends_on: `postgres`, `redis`
- **postgres**
  - image: `postgres:15-alpine`
  - env: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
  - volume: `postgres_data:/var/lib/postgresql/data`
- **redis**
  - image: `redis:7-alpine`
  - senha via `REDIS_PASSWORD`
  - volume: `redis_data:/data`

### Acesso

- UI: http://localhost:7860/

### Comandos úteis

```bash
# Subir serviços
docker compose up -d

# Logs do LangFlow
docker compose logs -f langflow

# Reiniciar serviço
docker compose restart langflow

# Encerrar e remover containers (mantém volumes)
docker compose down
```
