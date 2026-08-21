# Prefect Host

Docker Compose stack for a self-hosted [Prefect](https://www.prefect.io/) 3 server with Postgres and Redis.

## Services

| Service | Role |
|---------|------|
| `postgres` | Prefect API database (Postgres 14) |
| `redis` | Messaging backend |
| `prefect-server` | Prefect API / UI |
| `prefect-services` | Background server services |

Data is stored under `${PREFECT_DIR}` on the host:

- `postgresql/data` — database files
- `redis_data` — Redis persistence
- `prefect_data` — Prefect home
- `logs` — server logs

## Setup

```bash
cp .env.example .env
```

Edit `.env`:

| Variable | Description |
|----------|-------------|
| `PORT` | Host/container port for the Prefect API (default `4203`) |
| `PREFECT_DIR` | Host directory for persistent volumes |
| `PREFECT_API_URL` | Public API URL clients and the UI should use (e.g. `http://localhost:4203/api`) |

Ensure `PREFECT_DIR` exists and is writable by Docker:

```bash
mkdir -p "${PREFECT_DIR}"/{postgresql/data,redis_data,prefect_data,logs}
```

## Run

```bash
docker compose up -d
```

UI / API: `http://localhost:${PORT}` (API under `/api`).

Stop:

```bash
docker compose down
```

## Database migration

Postgres data lives in `${PREFECT_DIR}/postgresql/data`. Prefer a logical dump when moving hosts:

```bash
# Export (source)
docker exec prefect-postgres pg_dump -U prefect -Fc -d prefect -f /tmp/prefect.dump
docker cp prefect-postgres:/tmp/prefect.dump ./prefect.dump

# Import (target, empty Postgres 14+)
docker cp ./prefect.dump prefect-postgres:/tmp/prefect.dump
docker exec prefect-postgres pg_restore -U prefect -d prefect --clean --if-exists /tmp/prefect.dump
```

For a same-major-version move with downtime, you can stop the stack and `rsync` the `postgresql/data` directory instead. Do not copy a running data directory.

After migrate, update `PREFECT_API_URL` on the new host and start the stack.
