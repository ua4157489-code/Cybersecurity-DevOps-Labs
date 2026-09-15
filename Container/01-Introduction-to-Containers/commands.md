# Commands

## 1. Verify Operating System

```bash
cat /etc/os-release
```

## 2. Verify Podman

```bash
podman --version
```

## 3. Pull Hello World Image

```bash
podman pull hello-world
```

## 4. Run Hello World

```bash
podman run hello-world
```

## 5. List Containers

```bash
podman ps -a
```

## 6. Run Alpine Container

```bash
podman run -it alpine sh
```

## 7. Check Container Operating System

Run inside the container:

```bash
cat /etc/os-release
```

## 8. Check Container Hostname

Run inside the container:

```bash
hostname
```

## 9. Check Container Processes

Run inside the container:

```bash
ps
```

## 10. Exit Container

```bash
exit
```

## 11. Optional: List Local Images

```bash
podman images
```

## 12. Optional: List Containers

```bash
podman ps -a
```

---

# Command Summary

| Command               | Purpose                      |
| --------------------- | ---------------------------- |
| `cat /etc/os-release` | Identify operating system    |
| `podman --version`    | Verify Podman installation   |
| `podman pull`         | Download container image     |
| `podman run`          | Create and execute container |
| `podman ps -a`        | List containers              |
| `podman images`       | List local images            |
| `hostname`            | Display container hostname   |
| `ps`                  | Display container processes  |
| `exit`                | Leave interactive container  |
