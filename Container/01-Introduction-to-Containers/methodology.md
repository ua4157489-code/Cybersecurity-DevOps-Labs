# Methodology

## Lab: Introduction to Containers

### 1. Objective

The objective of this lab was to understand the fundamental concepts of containerization and gain practical experience running containers with Podman.

The lab focused on:

* Containerization fundamentals
* Container images
* Container execution
* Container isolation
* Process namespace isolation
* Container portability

---

## 2. Environment

The lab was performed on:

* **Operating System:** Ubuntu 24.04.3 LTS
* **Container Engine:** Podman 4.9.3
* **Architecture:** AMD64
* **User:** `ubuntu`
* **Container Images:** `hello-world`, `alpine`

---

## 3. Methodology

### Phase 1 — Environment Verification

The operating system and Podman installation were verified before starting the practical exercises.

```bash
cat /etc/os-release
podman --version
```

### Phase 2 — Container Image Retrieval

The official `hello-world` image was pulled from the container registry.

```bash
podman pull hello-world
```

### Phase 3 — Container Execution

The image was executed using:

```bash
podman run hello-world
```

Successful execution was confirmed from the returned output.

### Phase 4 — Container Verification

All containers were inspected using:

```bash
podman ps -a
```

The `Exited (0)` status confirmed successful execution.

### Phase 5 — Isolation Testing

An Alpine Linux container was launched interactively:

```bash
podman run -it alpine sh
```

The container's operating system, hostname, and processes were then inspected.

### Phase 6 — Process Isolation

The process list was examined using:

```bash
ps
```

The Alpine shell appeared as PID 1 inside the container, demonstrating the isolated process namespace.

### Phase 7 — Exit and Verification

The container was exited using:

```bash
exit
```

The terminal returned to the Ubuntu host environment.

---

## 4. Validation

The following conditions were used to determine successful completion:

1. Podman was installed and functional.
2. The `hello-world` image was successfully downloaded.
3. The container executed successfully.
4. The container exited with status code 0.
5. Alpine Linux ran independently from the Ubuntu host userspace.
6. The Alpine container had its own hostname.
7. The container presented its own process namespace.

---

## 5. Conclusion

The methodology demonstrated the complete basic container lifecycle:

**Image → Container → Execution → Verification → Isolation → Exit**

The practical results confirmed that Podman was functioning correctly and that containers provide isolated userspace and process environments while sharing the underlying Linux kernel.
