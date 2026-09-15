# Lab 2 — Findings

## 🎯 Overview

This document records the actual observations made while exploring the Podman CLI during Lab 2.

The findings are based on commands executed in the Ubuntu 24.04.3 LTS AWS EC2 environment running rootless Podman 4.9.3.

---

# 🔎 Finding 01 — Podman Environment

Podman was already installed and operational.

### Environment

```text
Operating System: Ubuntu 24.04.3 LTS
Podman: 4.9.3
Architecture: AMD64
Execution: Rootless
Runtime: runc
```

### Assessment

**PASS**

The environment was suitable for the container lifecycle exercises.

---

# 🔎 Finding 02 — Initial Container Inventory

The initial commands:

```bash
podman ps
podman ps -a
```

showed no existing containers.

### Assessment

**PASS**

The lab began from a clean container state.

---

# 🔎 Finding 03 — Alpine Container Execution

The following command successfully started Alpine:

```bash
podman run -it --name my_alpine alpine sh
```

Inside the container, `/etc/os-release` confirmed:

```text
Alpine Linux 3.24.1
```

The hostname was:

```text
6c5ae1c51715
```

### Assessment

**PASS**

The container executed successfully and provided an isolated shell environment.

---

# 🔎 Finding 04 — Container Main Process

The Alpine container used:

```text
sh
```

as its main process.

After:

```bash
exit
```

the container stopped.

### Assessment

This demonstrates an important container lifecycle concept:

**The container lifecycle is tied to the lifecycle of its main process.**

This was expected behavior.

---

# 🔎 Finding 05 — Restart Behavior

The following was tested:

```bash
podman restart my_alpine
```

The interactive Alpine container did not remain running because its original shell process had already exited.

A second test used:

```bash
podman run -d --name my_alpine_lifecycle alpine sleep 300
```

During restart testing, Podman reported a stop timeout and eventually required SIGKILL/force removal.

### Observed Message

```text
StopSignal SIGTERM failed to stop container
in 10 seconds, resorting to SIGKILL
```

### Assessment

**Environment-specific troubleshooting observation**

This behavior should not be described as a confirmed vulnerability.

---

# 🔎 Finding 06 — Container Removal

A container was created using:

```bash
podman create --name my_alpine alpine sh
```

and removed with:

```bash
podman rm my_alpine
```

The final:

```bash
podman ps -a
```

returned an empty list.

### Assessment

**PASS**

Container removal worked successfully.

---

# 🔎 Finding 07 — Short Image Name Resolution

The initial Nginx command was:

```bash
podman run -d --name nginx_container nginx
```

It failed with:

```text
short-name "nginx" did not resolve to an alias
and no unqualified-search registries are defined
```

### Cause

The Podman environment did not have an unqualified image search registry configured.

### Assessment

**Configuration limitation**

This is not a container vulnerability.

---

# 🔎 Finding 08 — Fully Qualified Image Reference

The following command succeeded:

```bash
podman run -d --name nginx_container docker.io/library/nginx
```

The image was successfully pulled and the container entered the running state.

### Assessment

**PASS**

Using a fully qualified image reference resolved the image lookup issue.

---

# 🔎 Finding 09 — Nginx Container Inspection

The Nginx container was inspected using:

```bash
podman inspect nginx_container
```

Important values included:

```text
Container ID:
26e1ac897f912234df838647d4f3cad593d29ef1c8e385fbba278c4bea6eca41

State:
running

Image:
docker.io/library/nginx:latest

Command:
nginx -g daemon off;

Runtime:
runc

Network:
slirp4netns
```

The container was not privileged.

### Assessment

**PASS**

The inspection successfully provided detailed container configuration and runtime information.

---

# 🔎 Finding 10 — Nginx Cleanup

The Nginx container was stopped and removed.

```bash
podman stop nginx_container
podman rm nginx_container
podman ps -a
```

The final container inventory was empty.

### Assessment

**PASS**

The lab completed with no remaining test containers.

---

# 📊 Overall Assessment

| Area                        | Result                         |
| --------------------------- | ------------------------------ |
| Podman environment          | ✅ Pass                         |
| Container listing           | ✅ Pass                         |
| Alpine execution            | ✅ Pass                         |
| Stop operation              | ✅ Pass                         |
| Restart test                | ⚠️ Troubleshooting observation |
| Container removal           | ✅ Pass                         |
| Nginx deployment            | ✅ Pass                         |
| Nginx inspection            | ✅ Pass                         |
| Nginx cleanup               | ✅ Pass                         |
| Image short-name resolution | ⚠️ Configuration issue         |

---

# 🧠 Key Takeaways

1. Container execution depends on the main process.
2. `podman ps` and `podman ps -a` provide different lifecycle views.
3. `podman inspect` is useful for configuration and security reviews.
4. Fully qualified image references avoid ambiguity when registry search is not configured.
5. Rootless Podman was successfully used throughout the lab.
6. Runtime behavior should be validated in the actual environment instead of assuming every container behaves identically.
