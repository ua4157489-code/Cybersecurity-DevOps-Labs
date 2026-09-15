# 🔧 Command Reference

## Lab 3 — Running Containers with Podman

This document contains the commands used to complete the practical exercises in this lab.

---

## 1. Verify Podman Environment

```bash
podman --version

podman info --format 'Rootless={{.Host.Security.Rootless}} | Runtime={{.Host.OCIRuntime.Name}} | Network={{.Host.NetworkBackend}}'

podman images
```

### Verified Environment

```text
Rootless=true
Runtime=runc
Network=netavark
```

---

## 2. Pull Nginx Image

```bash
podman pull docker.io/library/nginx:alpine
```

Verify the image:

```bash
podman images
```

---

## 3. Run Container in Detached Mode

```bash
podman run -d docker.io/library/nginx:alpine
```

Verify the running container:

```bash
podman ps
```

---

## 4. Configure Port Mapping

Stop existing running containers:

```bash
podman stop $(podman ps -q)
```

Run Nginx with host port `8080` mapped to container port `80`:

```bash
podman run -d -p 8080:80 docker.io/library/nginx:alpine
```

Verify:

```bash
podman ps
podman port $(podman ps -q)
```

Expected mapping:

```text
80/tcp -> 0.0.0.0:8080
```

Test the service:

```bash
curl http://localhost:8080
```

---

## 5. Configure Volume Mount

Create the host directory:

```bash
mkdir -p ~/nginx-content
```

Create test content:

```bash
echo "Hello from host!" > ~/nginx-content/index.html
```

Verify:

```bash
cat ~/nginx-content/index.html
```

Run the container with the bind mount:

```bash
podman run -d -p 8081:80 \
  -v ~/nginx-content:/usr/share/nginx/html:Z \
  docker.io/library/nginx:alpine
```

Test the mounted content:

```bash
curl http://localhost:8081
```

Expected:

```text
Hello from host!
```

Inspect the mount:

```bash
podman inspect <container_name> --format '{{json .Mounts}}'
```

---

## 6. Assign a Custom Container Name

```bash
podman run -d \
  --name my-nginx \
  -p 8082:80 \
  docker.io/library/nginx:alpine
```

Check the container status:

```bash
podman inspect my-nginx --format '{{.State.Status}}'
```

Verify port mapping:

```bash
podman port my-nginx
```

Test:

```bash
curl http://localhost:8082
```

---

## 7. Cleanup

Stop running containers:

```bash
podman stop $(podman ps -q)
```

Remove containers:

```bash
podman rm -f $(podman ps -aq)
```

Verify:

```bash
podman ps -a
podman images
```

Verify that host content remains:

```bash
cat ~/nginx-content/index.html
```

---

## ✅ Command Verification

| Task                            | Status      |
| ------------------------------- | ----------- |
| Podman environment verification | ✅ Completed |
| Nginx image pull                | ✅ Completed |
| Detached container              | ✅ Completed |
| Port mapping                    | ✅ Completed |
| Volume mount                    | ✅ Completed |
| Custom container name           | ✅ Completed |
| Container cleanup               | ✅ Completed |
