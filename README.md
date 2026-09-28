# n8n Compose Stack

Docker Compose configuration for running n8n with PostgreSQL, task runners, the n8n AI sandbox, and an internal SearXNG web-search service.

## Services

The active stack in `docker-compose.yaml` includes:

- **n8n** - workflow editor and execution service, available on port `5678`
- **PostgreSQL** - persistent n8n database
- **runners** - isolated task runners for n8n Code nodes
- **sandbox-api** - n8n AI sandbox service API
- **sandbox-runner-1** - privileged sandbox runner used by the AI sandbox
- **sandbox-certs** - one-time mTLS certificate bootstrap service
- **searxng** - internal web-search service used by n8n

The sandbox and SearXNG services are kept on the internal Compose network and are not published to the host.

## Prerequisites

- Docker Desktop or Docker Engine with the Compose plugin
- An n8n account or initial setup credentials created on first launch
- An API key for the configured AI model if AI features are enabled

## Quick start

1. Create the runtime environment file from the example:

   ```sh
   cp .env_example .env
   ```

2. Edit `.env` and replace every `change-me-*` value. Set the AI model API key in `N8N_INSTANCE_AI_MODEL_API_KEY` if you plan to use n8n AI features.

3. Start the stack in the background:

   ```sh
   docker compose up -d
   ```

4. Check service status and logs:

   ```sh
   docker compose ps
   docker compose logs -f n8n
   ```

5. Open n8n at <http://localhost:5678>.

## Configuration

`.env_example` contains the supported settings, including:

- `N8N_VERSION` and `N8N_SANDBOX_VERSION` for pinning compatible image versions
- Sandbox authentication and registration secrets
- `N8N_RUNNERS_AUTH_TOKEN` for n8n task runners
- SearXNG configuration
- AI model name and API key
- PostgreSQL database credentials

The sandbox API, runner, and sandbox images must use the same `N8N_SANDBOX_VERSION`.

Generate strong, unique values for all secrets. Do not commit `.env` or real API keys to version control.

## Common commands

```sh
# Start or recreate the stack
docker compose up -d

# Follow all logs
docker compose logs -f

# Follow one service
docker compose logs -f n8n

# Stop containers without deleting data
docker compose stop

# Stop and remove containers and the Compose network
docker compose down

# Validate the rendered configuration
docker compose config

# Update images, then recreate containers
docker compose pull
docker compose up -d
```

## Data and reset

Named volumes preserve application data when containers are removed:

- `n8n-data` stores n8n data and encryption-related state
- `db-storage` stores PostgreSQL data
- `sandbox-tls` stores generated sandbox certificates

To remove the stack and all stored data, use the following command carefully:

```sh
docker compose down -v
```

## Security notes

- Change all example secrets before starting the stack.
- Only the n8n port (`5678`) is published by the active Compose file.
- Do not publish sandbox or runner ports; the runner uses privileged Docker-in-Docker execution.
- For internet-facing deployments, configure TLS and a reverse proxy in front of n8n, and set the relevant webhook/editor base URLs in `.env`.
- Keep image versions pinned and update the n8n and sandbox versions together when required by their compatibility rules.

## Legacy configuration

`docker-compose_old.yaml` is an older, separate configuration that uses `n8n:latest`, PostgreSQL 16, and hard-coded demo settings. It is retained for reference and is not the recommended startup file.

To run it explicitly, provide the file name:

```sh
docker compose -f docker-compose_old.yaml up -d
```
