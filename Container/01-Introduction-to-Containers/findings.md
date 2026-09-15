# Findings

## Finding 01 — Podman Successfully Installed

**Status:** PASS

Podman 4.9.3 was available on the Ubuntu 24.04.3 LTS host.

```text
podman version 4.9.3
```

### Observation

The container engine was functional and ready to execute containers.

---

## Finding 02 — Container Image Successfully Retrieved

**Status:** PASS

The official `hello-world` image was successfully downloaded from Docker Hub.

```text
docker.io/library/hello-world:latest
```

### Observation

Podman successfully resolved and downloaded the required image.

---

## Finding 03 — Container Execution Successful

**Status:** PASS

The `hello-world` container executed successfully.

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Observation

The container was able to start, execute its process, produce output, and terminate normally.

---

## Finding 04 — Successful Container Exit

**Status:** PASS

The container status showed:

```text
Exited (0)
```

### Observation

Exit code `0` indicates successful execution without an application-level error.

---

## Finding 05 — Userspace Isolation

**Status:** PASS

The host operating system was Ubuntu 24.04.3 LTS, while the interactive container reported:

```text
NAME="Alpine Linux"
VERSION_ID=3.24.1
```

### Observation

The container provided its own userspace environment independently of the host distribution.

---

## Finding 06 — Hostname Isolation

**Status:** PASS

The Alpine container returned its own hostname:

```text
5c71bbb0ee34
```

### Observation

The container had an isolated hostname from the host environment.

---

## Finding 07 — Process Namespace Isolation

**Status:** PASS

The process list inside the container showed:

```text
PID   USER     TIME  COMMAND
1     root     0:00  sh
9     root     0:00  ps
```

### Observation

The shell appeared as PID 1 inside the container, demonstrating a separate process namespace.

---

# Overall Finding

The practical exercises successfully demonstrated basic containerization, image management, container execution, userspace isolation, hostname isolation, and process namespace isolation using Podman.
