# 🔎 Findings

## Lab 3 — Running Containers with Podman

## 1. Detached Container Execution

The Nginx Alpine image was successfully launched in detached mode.

```bash
podman run -d docker.io/library/nginx:alpine
```

The container was confirmed as running using:

```bash
podman ps
```

### Finding

Podman successfully ran the container as a background process.

**Status:** ✅ Successful

---

## 2. Port Mapping

The Nginx container port `80` was published on host port `8080`.

```text
80/tcp -> 0.0.0.0:8080
```

The service was tested using:

```bash
curl http://localhost:8080
```

The Nginx welcome page was returned successfully.

### Finding

Host-to-container port publishing was correctly configured and verified.

**Status:** ✅ Successful

---

## 3. Host Volume Mount

A host directory was mounted into the Nginx web root:

```text
/home/ubuntu/nginx-content
        ↓
/usr/share/nginx/html
```

The host file contained:

```text
Hello from host!
```

The same content was successfully returned from:

```bash
curl http://localhost:8081
```

### Finding

The bind mount allowed the container to serve content directly from the host filesystem.

**Status:** ✅ Successful

---

## 4. Custom Container Name

The container was created with the custom name:

```text
my-nginx
```

Its state was verified as:

```text
running
```

The service was exposed through:

```text
80/tcp -> 0.0.0.0:8082
```

### Finding

Custom container naming simplifies container identification and administration.

**Status:** ✅ Successful

---

## 5. Rootless Container Execution

The Podman environment reported:

```text
Rootless=true
```

### Finding

The lab was completed using rootless Podman execution.

This provides a reduced-privilege container-management model compared with running the container engine directly as root.

**Status:** ✅ Verified

---

## 6. Container Cleanup

After completing the practical exercises, all containers were removed.

```bash
podman ps -a
```

returned an empty container list.

The Nginx image remained available:

```bash
podman images
```

The host-mounted file also remained available:

```bash
cat ~/nginx-content/index.html
```

### Finding

Container cleanup removed the runtime resources while preserving the host filesystem data.

**Status:** ✅ Successful

---

# 📊 Findings Summary

| Area                    | Result       |
| ----------------------- | ------------ |
| Detached execution      | ✅ Successful |
| Port mapping            | ✅ Successful |
| HTTP connectivity       | ✅ Successful |
| Volume mounting         | ✅ Successful |
| Custom container naming | ✅ Successful |
| Rootless execution      | ✅ Verified   |
| Container cleanup       | ✅ Successful |
| Host data persistence   | ✅ Verified   |

---

# 🛡️ Security Observations

* Published ports should expose only services that are required.
* Bind mounts should be restricted to the minimum required host directories.
* Containers should preferably run without unnecessary privileges.
* Container images should be obtained from trusted registries and kept updated.
* Containers should be removed when they are no longer required.
