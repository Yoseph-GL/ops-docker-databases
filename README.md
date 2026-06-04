# Local DBaaS Sandbox

On-demand database platform for development, experimentation, and coursework.
One stack per directory, one `.env` per stack, one command to start.

## Architecture

All stacks share `core_network` (external bridge). Adminer and tooling reach any
database by container name without exposing additional ports.

| Stack | Engine | Host Port | Container Port | Persistence |
|---|---|---|---|---|
| `serious` | PostgreSQL 16.3 | `5432` | `5432` | Named volume |
| `playground` | PostgreSQL 16.3 | `5433` | `5432` | Named volume |
| `university` | MySQL 8.0 | `3306` | `3306` | Named volume |
| `test` | MySQL 8.0 | `3307` | `3306` | Ephemeral |
| `test` | PostgreSQL 16.3 | `5434` | `5432` | Ephemeral |
| `adminer` | Adminer 4.8 | `8082` | `8080` | Stateless |

All ports bound to `127.0.0.1`. Custom host ports avoid collisions with other
local database instances.

## Quickstart

```bash
cd serious/          # or playground/, university/, test/
cp .env.example .env
docker network create core_network 2>/dev/null || true
docker compose up -d
docker compose ps
```

## Adminer

```bash
cd adminer/
docker compose up -d
# http://db.test — server field = container name (e.g. serious_pg_v1)
```

## Shared Network

```bash
docker network create \
    --driver bridge \
    --label com.project=databases \
    --label com.description="Shared database bridge" \
    core_network
```

## Design

- **Restart policy** (`restart: 'no'`): containers started manually on demand.
  No background services consuming resources when idle.
- **Persistence:** `test` stacks are fully ephemeral (no volumes, data destroyed
  on `docker compose down`). All other stacks retain data via named Docker volumes.
