# Lab 14: Networking in Containers — Commands

## 1. List Podman Networks

```bash
podman network ls
```

```bash
podman network inspect podman
```

---

## 2. Create Custom Network

```bash
podman network create lab-network
```

```bash
podman network ls | grep lab-network
```

```bash
podman network inspect lab-network
```

---

## 3. Run Nginx with Port Publishing

```bash
podman run -d \
  --name webapp \
  -p 8080:80 \
  docker.io/library/nginx
```

Verify:

```bash
podman ps
```

Check port:

```bash
podman port webapp
```

---

## 4. Test Port Accessibility

```bash
curl -I http://localhost:8080
```

```bash
curl http://localhost:8080 | head -10
```

---

## 5. Attach Container to Custom Network

Stop the original container:

```bash
podman stop webapp
```

Remove it:

```bash
podman rm webapp
```

Run it on the custom network:

```bash
podman run -d \
  --name webapp \
  -p 8080:80 \
  --network lab-network \
  docker.io/library/nginx
```

---

## 6. Inspect Network Assignment

```bash
podman inspect webapp --format '{{.HostConfig.NetworkMode}}'
```

```bash
podman inspect webapp --format '{{json .NetworkSettings.Networks}}'
```

```bash
podman port webapp
```

---

## 7. Final Connectivity Test

```bash
curl -I http://localhost:8080
```

```bash
curl http://localhost:8080 | head -5
```

---

## 8. Cleanup

```bash
podman stop webapp
podman rm webapp
podman network rm lab-network
```
