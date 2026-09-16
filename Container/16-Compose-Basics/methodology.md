# 🧪 Methodology — Lab 16

## 1. Scope

This lab evaluates multi-container application management using Podman Compose and focuses on:

- Service definition with `podman-compose.yml`
- Multi-container deployment (Nginx + PostgreSQL)
- Port publishing
- Bind-mount behavior
- Compose network creation and inspection
- Service health validation
- Cleanup behavior verification

## 2. Preparation

The lab was performed from:

```
~/Alrazzaq_Labs/Container/16-Compose-Basics
```

Podman and `podman-compose` versions were verified before deployment:

```bash
podman --version
podman-compose --version
```

## 3. Compose Definition

A `podman-compose.yml` file was created with two services:

- `web` using `nginx:alpine`
- `db` using `postgres:13`

The `web` service was configured with port publishing (`8080:80`) and a bind mount (`./html:/usr/share/nginx/html`). PostgreSQL was configured with a lab-only password via the `POSTGRES_PASSWORD` environment variable.

## 4. Application Deployment

The application was started with:

```bash
podman-compose up -d
```

The resulting containers were:

```
16-compose-basics_web_1
16-compose-basics_db_1
```

## 5. Service Validation

Nginx was tested through the published host port:

```bash
curl -i http://localhost:8080
```

PostgreSQL was validated with `pg_isready` and a SQL version query executed inside the running container.

## 6. Network Validation

The automatically created Compose network was inspected with:

```bash
podman network inspect 16-compose-basics_default
```

The network used the `bridge` driver with subnet `10.89.1.0/24` and gateway `10.89.1.1`.

## 7. Volume / Bind-Mount Validation

The host HTML file was modified while the container was running. The updated content was immediately served by Nginx, demonstrating the live behavior of the bind mount without requiring an image rebuild or container restart.

## 8. Cleanup

The Compose application was stopped with:

```bash
podman-compose down
```

The containers were removed. The Compose network remained in this environment and was verified separately rather than assumed to be removed.

## 9. Evidence Standard

Only results actually observed during the lab are documented. Where a command's behavior (such as network removal on `podman-compose down`) did not match expectations, the actual observed behavior was recorded rather than the assumed behavior.
