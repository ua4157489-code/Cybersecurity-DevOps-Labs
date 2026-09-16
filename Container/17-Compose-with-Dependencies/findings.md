# Lab 17 — Findings

## 1. Executive Summary

The lab successfully demonstrated service dependency management, container networking, service discovery, Redis-backed application state, and application scaling using Podman and Podman Compose.

The final environment successfully supported three Flask application replicas connected to a shared Redis backend.

---

## 2. Finding: Service Dependency

### Observation

The Flask application was configured with Redis as a dependency:

```yaml
depends_on:
  - redis
```

### Result

The Redis service was available as the backend dependency for the Flask application.

### Impact

Correct dependency configuration helps establish the intended startup relationship between application and backend services.

### Status

**Verified**

---

## 3. Finding: Redis Service Connectivity

### Observation

The Flask health endpoint returned:

```text
Redis connection OK
```

Redis itself responded to:

```text
PING
```

with:

```text
PONG
```

### Impact

This confirms that the application successfully communicated with the Redis service.

### Status

**Verified**

---

## 4. Finding: Shared Application State

### Observation

Three requests to the Flask application returned:

```text
1
2
3
```

The Redis database independently reported:

```text
3
```

for the `hits` key.

### Impact

This confirms that the application was storing shared state in Redis rather than maintaining the counter only inside the Flask process.

### Status

**Verified**

---

## 5. Finding: Container Service Discovery

### Observation

The Flask container resolved:

```text
redis
```

to:

```text
10.89.0.6
```

using container DNS.

### Impact

Service-name-based communication avoids depending on static container IP addresses.

### Status

**Verified**

---

## 6. Finding: Dedicated Compose Network

### Observation

Podman created:

```text
17-compose-with-dependencies_default
```

with:

```text
Driver:    bridge
Subnet:    10.89.0.0/24
Gateway:   10.89.0.1
DNS:       enabled
```

### Impact

The dedicated network provides isolated communication between the services participating in the Compose application.

### Status

**Verified**

---

## 7. Finding: Application Scaling

### Observation

Three Flask application containers were successfully running:

```text
17-compose-with-dependencies_webapp_1
17-compose-with-dependencies_webapp_2
17-compose-with-dependencies_webapp_3
```

The containers had unique identifiers:

```text
144b3f780888
bb7642b4ca5a
d8ae9f3ca579
```

### Impact

Multiple application instances can operate simultaneously and share the same Redis backend.

### Status

**Verified**

---

## 8. Finding: Replica-to-Redis Connectivity

### Observation

All three Flask replicas returned:

```text
True
```

when tested against Redis.

### Impact

Every application replica was able to reach the shared backend service.

### Status

**Verified**

---

## 9. Troubleshooting Finding: Missing requirements.txt

### Observation

The initial container build failed because:

```text
requirements.txt
```

was missing.

### Resolution

The file was created with:

```text
flask
redis
```

The image was then rebuilt successfully.

### Status

**Resolved**

---

## 10. Troubleshooting Finding: Incorrect COPY Path

### Observation

The initial Containerfile attempted to copy:

```dockerfile
COPY app.py .
```

However, the application was located at:

```text
app/app.py
```

### Resolution

The Containerfile was corrected to:

```dockerfile
COPY app/app.py .
```

### Status

**Resolved**

---

## 11. Scaling Observation

### Observation

The initial `podman-compose --scale` attempt did not complete normally and became stuck while stopping the existing webapp container.

### Resolution

The existing container was removed and the additional application replicas were started directly with Podman on the same Compose network.

### Result

Three independent Flask containers were successfully running and all three successfully connected to Redis.

### Status

**Resolved**

---

# 12. Overall Findings

| Area          | Finding                              | Status   |
| ------------- | ------------------------------------ | -------- |
| Dependencies  | Redis configured as Flask dependency | Verified |
| Application   | Flask service operational            | Verified |
| Backend       | Redis operational                    | Verified |
| Networking    | Dedicated bridge network             | Verified |
| DNS           | Redis service-name resolution        | Verified |
| Data          | Shared Redis counter                 | Verified |
| Scaling       | Three Flask replicas                 | Verified |
| Connectivity  | All replicas reach Redis             | Verified |
| Build issue   | Missing requirements file            | Resolved |
| Build issue   | Incorrect application COPY path      | Resolved |
| Scaling issue | Compose scaling interruption         | Resolved |

---

# 13. Conclusion

The lab successfully demonstrated the practical operation of a multi-container application with a shared Redis backend.

The final configuration provided:

```text
1 Redis service
        +
3 Flask application replicas
        +
1 dedicated Compose network
        +
DNS-based service discovery
        +
shared Redis application state
```

All major functional and connectivity tests passed.

