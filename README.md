# Home System

Docker Compose stacks for home services:

- Actual Budget
- n8n
- InvenTree

All stacks live under `projects/<name>/edge` and can be managed from anywhere with `docker compose -f ...`.

## Requirements

- Docker Engine + Docker Compose plugin (`docker compose`)
- A host with ports available for each stack

## Project Names and Multi-Stack Hosts

Use Compose project names so multiple stacks can run on the same host without clashing on network/volume names.

Two ways to set the project name:

1. Put `COMPOSE_PROJECT_NAME=...` in the stack env file
2. Pass `-p <name>` on commands (this overrides `COMPOSE_PROJECT_NAME`)

Examples:

```powershell
# Explicit project names (recommended for automation)
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env up -d
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env up -d
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env up -d
```

Note: the current compose files set fixed `container_name` values, which means you cannot run two copies of the same stack on one host without editing/removing those `container_name` fields.

## Quick Start (All Projects)

```powershell
# from repository root
Copy-Item projects/actual/edge/actual.env.example projects/actual/edge/actual.env -ErrorAction SilentlyContinue
Copy-Item projects/n8n/edge/n8n.env.example projects/n8n/edge/n8n.env -ErrorAction SilentlyContinue
Copy-Item projects/inventree/edge/.env.example projects/inventree/edge/.env -ErrorAction SilentlyContinue
```

Then edit each env file before first run.

## Actual Budget

Path: `projects/actual/edge`

### Configure

1. Create `actual.env` from `actual.env.example`
2. Set `ACTUAL_HOST_PORT` (default `5006`)

### Start

```powershell
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env up -d
```

### Manage

```powershell
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env ps
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env logs -f
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env pull
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env up -d
docker compose -p hs-actual -f projects/actual/edge/docker-compose.yml --env-file projects/actual/edge/actual.env down
```

### Access

- `http://<host>:<ACTUAL_HOST_PORT>`

## n8n

Path: `projects/n8n/edge`

### Configure

1. Create `n8n.env` from `n8n.env.example`
2. Set at least:
	- `N8N_HOST`
	- `N8N_PROTOCOL`
	- `N8N_BASIC_AUTH_USER`
	- `N8N_BASIC_AUTH_PASSWORD`

### Start

```powershell
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env up -d
```

### Manage

```powershell
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env ps
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env logs -f
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env pull
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env up -d
docker compose -p hs-n8n -f projects/n8n/edge/docker-compose.yml --env-file projects/n8n/edge/n8n.env down
```

### Access

- `http://<host>:5678`

## InvenTree

Path: `projects/inventree/edge`

### Configure

1. Create `.env` from `.env.example`
2. Update at least:
	- `INVENTREE_DB_PASSWORD`
	- `INVENTREE_SITE_URL` (host only, e.g. `http://debgosa`)
	- `INVENTREE_HTTP_PORT` / `INVENTREE_HTTPS_PORT` if needed

Important:
With the current `Caddyfile`, do not include a custom port in `INVENTREE_SITE_URL`.
Use `INVENTREE_HTTP_PORT` and `INVENTREE_HTTPS_PORT` to control exposed host ports.

### First-Time Database Setup

Run database migrations before first start:

```powershell
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env run --rm inventree-server invoke update
```

Create the admin account (interactive command):

```powershell
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env run --rm inventree-server invoke superuser
```

If you want non-interactive first-run bootstrap, set `INVENTREE_ADMIN_USER`,
`INVENTREE_ADMIN_PASSWORD`, and `INVENTREE_ADMIN_EMAIL` in `.env` and run
`invoke update`, then remove those values from `.env`.

### Start

```powershell
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env up -d
```

### Manage

```powershell
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env ps
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env logs -f
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env pull
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env up -d
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env down
```

### Access

- `http://<host>:<INVENTREE_HTTP_PORT>`

## Shared Operations

Use the same command shape for any stack:

```powershell
docker compose -p <project-name> -f <compose-file> --env-file <env-file> <command>
```

Useful commands:

- `config` validate interpolated compose config
- `ps` list containers in a stack
- `logs -f` follow logs
- `pull` pull new images
- `up -d` apply updates and (re)start
- `down` stop and remove stack resources

## Troubleshooting

- `docker` command not found: install Docker Desktop / Docker Engine and ensure `docker` is on `PATH`
- Port conflicts: change host ports in env files before `up`
- Validate config before deploy:

```powershell
docker compose -p hs-inventree -f projects/inventree/edge/docker-compose.yml --env-file projects/inventree/edge/.env config
```
