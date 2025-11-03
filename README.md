# PostgreSQL via Docker (Windows)

This repository contains instructions and examples for running PostgreSQL using Docker on Windows. It covers quick start commands, persistent data, PowerShell-friendly examples, and common troubleshooting tips.

## Prerequisites

- Docker Desktop installed and running on Windows
- (Optional) psql client installed locally (or use the client inside the container)

## Quick start — run a temporary PostgreSQL container

Run a quick container for testing (this does NOT persist data across container removal):

```powershell
docker run --name my_postgres -e POSTGRES_USER=username -e POSTGRES_PASSWORD=password -e POSTGRES_DB=database_name -p 5432:5432 -d postgres
```

Check it's running:

```powershell
docker ps
docker logs my_postgres
```

Connect with psql from your host (replace values as needed):

```powershell
psql -h localhost -p 5432 -U username -d database_name
```

## Create with persistent data (recommended)

Use a named Docker volume so the database files survive container restarts and recreations:

```powershell
docker run --name my_postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_DB=mydb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  -d postgres
```

Alternative: map a host directory (Windows) to the container. On Docker Desktop, ensure the folder is shared (Settings → Resources → File Sharing) or use WSL paths. Example mapping:

```powershell
# Example (adjust path and permissions first):
docker run --name my_database \
  -e POSTGRES_USER=username \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=database_name \
  -v G:\WSL\pgdata\var\lib\postgresql\data:/var/lib/postgresql/data \
  -p 5432:5432 \
  -d postgres
```

Notes:
- Mapping to Windows drives can run into permission or path translation issues. If possible, prefer Docker named volumes for reliability.
- When using WSL2, mapping a WSL path or creating the data directory inside the WSL filesystem is often the most stable approach.

## Start / Stop / Restart

Start an existing container:

```powershell
docker container start my_postgres
```

Stop:

```powershell
docker container stop my_postgres
```

Restart (container):

```powershell
docker container restart my_postgres
```

If you installed PostgreSQL natively on Windows (outside of Docker), you may use `pg_ctl` for the local service. Example (native install):

```powershell
pg_ctl -D "C:\Program Files\PostgreSQL\18\data" restart
```

## Inject SQL from the host into the container

Run a single command from the host into the container (useful for quick inserts or schema changes):

```powershell
docker exec -it my_database psql -U username -d database_name -c "INSERT INTO product (product_name, price) VALUES ('Watch', 40);"
```

You can also copy and run a .sql file:

```powershell
docker cp ./init.sql my_database:/init.sql
docker exec -it my_database psql -U username -d database_name -f /init.sql
```

## Example SQL

```sql
CREATE TABLE product (
  product_name VARCHAR(100),
  price INTEGER
);

INSERT INTO product (product_name, price) VALUES ('Pants', 100);
SELECT * FROM product;
```

## Troubleshooting

- If you cannot connect from the host to the container, ensure the container is listening on the forwarded port (check `docker ps` and `docker logs`).
- If you get permission errors for a host-mounted data directory, check Docker Desktop file sharing settings or use a named Docker volume instead.
- To inspect container logs:

```powershell
docker logs my_postgres
```

- To run an interactive shell inside the container for debugging:

```powershell
docker exec -it my_postgres bash
```

## Security notes

- Do not use easily guessable passwords for production systems. Consider secrets managers or environment file mechanisms for more secure handling of credentials.
- For production, configure additional options (pg_hba.conf, SSL, network restrictions) and use a proper backup strategy.

## Useful Docker commands

- List containers (running): `docker ps`
- List all containers: `docker ps -a`
- Remove a container: `docker rm <container>`
- Remove a volume: `docker volume rm <volume>`

## Where this came from
This README consolidates the quick steps provided in the original notes and adds a few clarifications for Windows and Docker Desktop users.

---

If you want, I can also:
- add a small `docker-compose.yml` for local development,
- add an `init.sql` example and show how to mount/run it at container startup,
- or create a PowerShell script that automates the run/create/start steps.

Author: nbpacio and GPT5
