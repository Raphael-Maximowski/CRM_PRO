# CRM PRO — Wrapper de Orquestração

Este repositório é um **wrapper** que reúne e orquestra todos os serviços do CRM PRO através do Docker Compose. Ele não contém o código da aplicação diretamente — o backend e o frontend são incluídos como **submódulos Git**.

## Arquitetura

O `docker-compose.yml` sobe três serviços:

| Serviço    | Descrição                  | Porta (host) | Tecnologia        |
|------------|----------------------------|--------------|-------------------|
| `postgres` | Banco de dados             | `5432`       | PostgreSQL 16     |
| `api`      | Backend / API              | `3000`       | NestJS + Prisma   |
| `web`      | Frontend                   | `3001`       | Next.js           |

Submódulos:

| Caminho            | Repositório                                  |
|--------------------|----------------------------------------------|
| `packages/backend` | `git@github.com:Alan-pereiraa/CRM_PRO_API.git` |
| `platform/web`     | `git@github.com:Alan-pereiraa/CRM_PRO.git`     |

## Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/) (já incluso no Docker Desktop)
- [Git](https://git-scm.com/)
- Acesso SSH configurado no GitHub (os submódulos usam URLs `git@github.com`)

## 1. Clonar o repositório com os submódulos

Como o projeto usa submódulos, clone com a flag `--recurse-submodules`:

```bash
git clone --recurse-submodules git@github.com:Raphael-Maximowski/CRM_PRO.git
cd CRM_PRO
```

Se já clonou sem a flag, inicialize os submódulos manualmente:

```bash
git submodule update --init --recursive
```

## 2. Configurar variáveis de ambiente

Copie o arquivo de exemplo e ajuste os valores conforme necessário:

```bash
cp .env.example .env
```

Variáveis disponíveis:

| Variável            | Descrição                          | Padrão                                |
|---------------------|------------------------------------|---------------------------------------|
| `POSTGRES_USER`     | Usuário do banco                   | `postgres`                            |
| `POSTGRES_PASSWORD` | Senha do banco                     | `postgres`                            |
| `POSTGRES_DB`       | Nome do banco                      | `crm_pro_db`                          |
| `JWT_SECRET`        | Segredo para tokens JWT            | *(troque por um valor forte)*         |
| `SECRET_KEY`        | Chave secreta da aplicação         | *(troque por um valor forte)*         |

> **Importante:** em produção, substitua `JWT_SECRET` e `SECRET_KEY` por valores aleatórios e seguros.

## 3. Subir o ambiente

```bash
docker compose up --build
```

- `--build` força a reconstrução das imagens (use na primeira execução ou após mudar dependências).
- Para rodar em segundo plano (modo detached): `docker compose up --build -d`.

O Postgres tem _healthcheck_ — a API só inicia depois que o banco estiver pronto.

Na inicialização do container da API, o entrypoint executa automaticamente:

1. `npx prisma generate` — gera o cliente Prisma;
2. `npx prisma migrate deploy` — aplica as migrações no banco;
3. `npm run start:dev` — sobe o NestJS em modo _watch_.

Ou seja, **não é necessário rodar as migrações manualmente** — basta subir o ambiente.

## 4. Acessar a aplicação

| Recurso  | URL                       |
|----------|---------------------------|
| Frontend | http://localhost:3001     |
| API      | http://localhost:3000     |
| Postgres | `localhost:5432`          |

## Estrutura do projeto

```
CRM/
├── docker-compose.yml      # Orquestração dos serviços
├── .env.example            # Modelo de variáveis de ambiente
├── .gitmodules             # Definição dos submódulos
├── packages/
│   └── backend/            # Submódulo: API (NestJS + Prisma)
└── platform/
    └── web/                # Submódulo: Frontend (Next.js)
```
