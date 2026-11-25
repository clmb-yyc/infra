# clmb local infrastructure

Run the shared development services from sibling checkouts:

```text
clmb-yyc/
  infra/          # this repository
  atelier/
    api/
```

Install Docker with Compose, start Docker, and clone `atelier` and `infra` with these directory names. Compose resolves the API build context relative to `compose.yaml`, regardless of your current working directory.

## Setup and startup

1. Copy `.env.example` to `.env` in `infra` and set a local database password.
2. Generate a Base64-encoded JWT key from at least 32 random bytes, and set `ATELIER_JWT_SECRET` in `.env`. For example, with OpenSSL:

   ```sh
   openssl rand -base64 32
   ```

   In PowerShell, an alternative is:

   ```powershell
   $bytes = New-Object byte[] 32
   [System.Security.Cryptography.RandomNumberGenerator]::Fill($bytes)
   [Convert]::ToBase64String($bytes)
   ```

3. From `infra`, run:

   ```sh
   docker compose up --build -d
   docker compose logs -f
   ```

The API waits for PostgreSQL to become healthy. After API source changes, rebuild with `docker compose up --build -d api`. Stop the stack with `docker compose down`.

## Connections

| Service | From the host | From Compose containers |
| --- | --- | --- |
| API | `http://127.0.0.1:8080` | `http://api:8080` |
| PostgreSQL | `127.0.0.1:5433`, database `atelier` | `postgres:5432`, database `atelier` |
| Kafka | `localhost:19092` | `kafka:9092` |

Use the database credentials from `.env`. Kafka is available for development, but the API does not yet publish or consume events. Future service repositories can be checked out alongside `infra` and added to Compose with sibling build paths such as `../service`.

## Persistent data

The project name `clmb` gives the named volumes `clmb_postgres-data` and `clmb_kafka-data`. `docker compose down` preserves them; `docker compose down -v` deletes their data. PostgreSQL initializes its user and password only on an empty volume: editing `.env` does not change credentials in an existing database. Update the database credentials separately or remove the volume if its data is disposable.

Kafka persists its cluster identity in its volume. This configuration uses cluster ID `MkU3OEVBNTcwNTJENDM2Qk`; if an existing Kafka volume was initialized with another ID, set `CLUSTER_ID` in `compose.yaml` to that existing ID or remove the volume if its data is disposable.
