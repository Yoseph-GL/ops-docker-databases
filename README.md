# Local DBaaS Sandbox

On-demand database platform for development, experimentation, and coursework.
KISS philosophy: one stack per directory, one `.env` per stack, one command to start.

## Architecture

All stacks share `core_network` (external bridge) so adminer and other tooling can
reach any database by container name without exposing additional ports.

| Stack | Engine | Host Port | Container Port | Persistence |
|---|---|---|---|---|
| `serious` | PostgreSQL 16.3 | `5432` | `5432` | Named volume |
| `playground` | PostgreSQL 16.3 | `5433` | `5432` | Named volume |
| `university` | MySQL 8.0 | `3306` | `3306` | Named volume |
| `test` | MySQL 8.0 | `3307` | `3306` | None (ephemeral) |
| `test` | PostgreSQL 16.3 | `5434` | `5432` | None (ephemeral) |
| `adminer` | Adminer 4.8 | `8082` | `8080` | Stateless |

All ports bound to `127.0.0.1` — no LAN exposure. Custom host ports avoid
collisions with other local database instances and allow simultaneous access
via DataGrip, DBeaver, or psql.

## Quick Start

```bash
cd serious/          # or playground/, university/, test/

cp .env.example .env # generate credentials
docker network create core_network 2>/dev/null || true
docker compose up -d
docker compose ps
```

## Adminer (SQL GUI)

```bash
cd adminer/
docker compose up -d
# Open http://db.test — server field = container name (e.g. serious_pg_v1)
```

## Shared Network

```bash
docker network create \
    --driver bridge \
    --label com.project=databases \
    --label com.description="Shared database bridge" \
    core_network
```

## Design Decisions

**Restart policy (`restart: 'no'`):** Containers are started manually on demand.
No background services consuming CPU or RAM when not in use — relevant for
battery-constrained laptop development.

**Persistence model:** `test` is the only fully ephemeral stack — no volumes
mounted, data destroyed on `docker compose down`. Designed for CI-like workflows
and throwaway integration tests. All other stacks retain data across container
recreation via named Docker volumes.
