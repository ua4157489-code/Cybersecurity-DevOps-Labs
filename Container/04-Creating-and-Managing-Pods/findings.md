# 🔎 Findings — Lab 4

## 1. Executive Summary

The Podman pod deployment was partially successful.

The core pod functionality was verified using Nginx and Redis. Nginx successfully served HTTP traffic through the pod port mapping, Redis returned `PONG`, and Nginx successfully responded through the pod's shared network namespace.

The shared-volume test exposed a container-state issue: the additional `nginx2` and `redis2` containers did not remain running. Consequently, the file-sharing test could not be completed.

## 2. Finding: Rootless Podman Environment

**Status:** Successful

Observed environment:

- Podman 4.9.3
- Rootless=true
- Runtime=runc
- Network=netavark

The lab was executed using rootless Podman.

## 3. Finding: Pod Creation

**Status:** Successful

Command:

```bash
podman pod create --name demo-pod -p 8080:80
```

The `demo-pod` pod was successfully created.

## 4. Finding: Nginx Deployment

**Status:** Successful

Nginx was started inside the pod:

```bash
podman run -d --pod demo-pod --name nginx-container docker.io/library/nginx:alpine
```

The container remained running.

## 5. Finding: Host Port Mapping

**Status:** Successful

The pod was created with:

```
8080:80
```

Testing:

```bash
curl http://localhost:8080
```

returned the standard Nginx welcome page.

## 6. Finding: Redis Deployment

**Status:** Successful

Redis was started inside the same pod:

```bash
podman run -d --pod demo-pod --name redis-container docker.io/library/redis:alpine
```

Validation:

```bash
podman exec redis-container redis-cli ping
```

Result:

```
PONG
```

## 7. Finding: Shared Pod Networking

**Status:** Successful

Nginx was accessed through localhost from inside the container:

```bash
podman exec nginx-container sh -c 'wget -qO- http://127.0.0.1:80 | head'
```

The successful response confirmed the expected shared network namespace behavior of the pod.

## 8. Finding: Shared Volume Creation

**Status:** Successful

The named volume was created:

```bash
podman volume create shared-vol
```

The volume was successfully inspected with:

```bash
podman volume inspect shared-vol
```

## 9. Finding: Volume-Mounted Containers

**Status:** Failed / Requires Investigation

The following containers were launched:

```bash
podman run -d --pod demo-pod --name nginx2 -v shared-vol:/data docker.io/library/nginx:alpine
podman run -d --pod demo-pod --name redis2 -v shared-vol:/data docker.io/library/redis:alpine
```

They did not remain running.

Attempting to execute commands inside them produced:

```
Error: can only create exec sessions on running containers: container state improper
```

Therefore, the expected file-sharing verification was not completed.

## 10. Finding: Pod Degraded State

**Status:** Observed

After the additional containers failed to remain running, the pod status became:

```
Degraded
```

This is consistent with a pod containing containers that are no longer running.

## 11. Finding: Cleanup

**Status:** Successful

The pod was removed with:

```bash
podman pod rm -f demo-pod
```

The volume was removed with:

```bash
podman volume rm shared-vol
```

Final checks:

```bash
podman pod ps
podman ps -a
podman volume ls
```

No remaining lab resources were observed.

## 12. Conclusion

The primary pod, networking, Nginx, Redis, and port-mapping objectives were successfully demonstrated.

The shared-volume objective requires additional troubleshooting because the volume-mounted containers exited before the file-sharing test could be performed.
