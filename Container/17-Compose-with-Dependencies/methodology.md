# Lab 17 — Methodology

## 1. Purpose

The purpose of this lab was to demonstrate practical container orchestration concepts using Podman Compose.

The lab focused on:

* Service dependencies
* Container networking
* Service discovery
* Redis-backed application state
* Multi-container application deployment
* Application scaling
* Inter-container connectivity

---

## 2. Lab Architecture

The environment consisted of:

```text
                 Podman Compose Network
                    10.89.0.0/24
                          |
              +-----------+-----------+
              |                       |
            Redis                  Flask Webapp
            :6379                     :5000
              |                       |
              |              +--------+--------+
              |              |        |        |
              |           webapp_1 webapp_2 webapp_3
              |              |        |        |
              +--------------+--------+--------+
```

Redis was used as the shared backend database.

The Flask application used the Redis service name:

```text
redis
```

rather than a hard-coded IP address.

---

## 3. Dependency Management

The Compose configuration defined:

```yaml
depends_on:
  - redis
```

This establishes Redis as a dependency of the Flask web application.

Podman Compose created the webapp with a dependency requirement for the Redis container.

This demonstrated the relationship between application and backend services.

---

## 4. Application Deployment

The Flask application was packaged into a container image based on:

```text
python:3.12-alpine
```

The application dependencies were:

```text
flask
redis
```

The container exposed port:

```text
5000
```

The application listened on:

```text
0.0.0.0:5000
```

---

## 5. Redis Integration

The application configured Redis using:

```python
redis_host = os.environ.get("REDIS_HOST", "redis")
```

The Compose configuration supplied:

```yaml
environment:
  REDIS_HOST: redis
```

This allowed the application to locate Redis through Compose DNS.

---

## 6. Functional Testing

The health endpoint was tested:

```bash
curl http://localhost:5000/health
```

The application returned:

```text
Redis connection OK
```

The root endpoint was then accessed three times.

The Redis-backed counter returned:

```text
1
2
3
```

The value was independently confirmed using:

```bash
podman exec 17-compose-with-dependencies_redis_1 redis-cli GET hits
```

Result:

```text
3
```

This provided both application-level and backend-level verification.

---

## 7. Network Verification

The Compose network was inspected.

The actual network configuration was:

```text
Network:    17-compose-with-dependencies_default
Driver:     bridge
Interface:  podman1
Subnet:     10.89.0.0/24
Gateway:    10.89.0.1
DNS:        enabled
```

Redis service discovery was tested from the Flask container:

```bash
getent hosts redis
```

The service resolved to:

```text
10.89.0.6
```

This demonstrated functional container DNS.

---

## 8. Scaling Method

The initial Flask Compose service published host port `5000`.

Since multiple containers cannot simultaneously bind the same host port, a separate scaling configuration was used without host port publishing.

Three webapp containers were then attached to the same Compose network:

```text
webapp_1
webapp_2
webapp_3
```

Each replica was independently verified.

---

## 9. Replica Connectivity Testing

Each replica was tested using the Python Redis client.

The test performed:

1. Read `REDIS_HOST`.
2. Connect to Redis on port `6379`.
3. Execute Redis `PING`.
4. Return the result.

All three replicas returned:

```text
True
```

Therefore, all replicas successfully communicated with the same Redis service.

---

## 10. Evidence Collection

Five screenshots were selected as the primary lab evidence:

```text
01-compose-dependencies-config.png
02-running-services.png
03-flask-redis-functionality.png
04-network-and-service-discovery.png
05-scaled-replicas-connectivity.png
```

The evidence covers:

* Configuration
* Running containers
* Application functionality
* Network/service discovery
* Scaling and replica connectivity

---

## 11. Validation Criteria

The lab was considered successful when the following conditions were met:

| Validation                 | Result |
| -------------------------- | ------ |
| Redis container running    | Passed |
| Flask container running    | Passed |
| `depends_on` configured    | Passed |
| Flask health check         | Passed |
| Redis `PING`               | Passed |
| Redis counter              | Passed |
| Compose network created    | Passed |
| Redis DNS resolution       | Passed |
| Three webapp replicas      | Passed |
| Replica 1 Redis connection | Passed |
| Replica 2 Redis connection | Passed |
| Replica 3 Redis connection | Passed |

---

## 12. Completion

All defined objectives were successfully demonstrated.

The final environment contained one Redis service and three Flask web application containers connected through the same Podman network.
