# Lab 2 — Exploring Podman CLI — Command Reference

## 🎯 Purpose

This document contains the commands actually used during Lab 2 to explore Podman container lifecycle management, inspection, troubleshooting, and cleanup.

---

# 1. Environment Verification

## Check Operating System

```bash
cat /etc/os-release
```

## Check Podman Version

```bash
podman --version
```

---

# 2. List Containers

## Running Containers

```bash
podman ps
```

## All Containers

```bash
podman ps -a
```

---

# 3. Run Alpine Container

## Start Interactive Container

```bash
podman run -it --name my_alpine alpine sh
```

## Check Alpine Version Information

Run inside the container:

```bash
cat /etc/os-release
```

## Check Container Hostname

```bash
hostname
```

## Exit

```bash
exit
```

---

# 4. Stop Container

```bash
podman stop my_alpine
```

## Verify

```bash
podman ps -a
```

---

# 5. Restart Container

```bash
podman restart my_alpine
```

## Verify Running Containers

```bash
podman ps
```

## Verify All Containers

```bash
podman ps -a
```

The interactive Alpine container did not remain running because its main `sh` process had already terminated.

---

# 6. Test Long-Running Container

```bash
podman run -d --name my_alpine_lifecycle alpine sleep 300
```

## Verify

```bash
podman ps
```

## Restart

```bash
podman restart my_alpine_lifecycle
```

## Check State

```bash
podman ps -a
```

## Force Remove

```bash
podman rm -f my_alpine_lifecycle
```

## Migrate Podman State

```bash
podman system migrate
```

## Final Check

```bash
podman ps -a
```

---

# 7. Create a Container Without Starting It

```bash
podman create --name my_alpine alpine sh
```

## Verify

```bash
podman ps -a
```

---

# 8. Remove Container

```bash
podman rm my_alpine
```

## Verify

```bash
podman ps -a
```

---

# 9. Nginx Image Resolution Test

## Short Image Name

```bash
podman run -d --name nginx_container nginx
```

This failed because the environment did not have an unqualified search registry configured.

---

# 10. Nginx Using Fully Qualified Image

```bash
podman run -d --name nginx_container docker.io/library/nginx
```

## Verify

```bash
podman ps
```

---

# 11. Inspect Nginx

## Full Inspection

```bash
podman inspect nginx_container
```

## Container ID

```bash
podman inspect --format '{{.Id}}' nginx_container
```

## Container State

```bash
podman inspect --format '{{.State.Status}}' nginx_container
```

## Image

```bash
podman inspect --format '{{.Config.Image}}' nginx_container
```

## Command

```bash
podman inspect --format '{{.Config.Cmd}}' nginx_container
```

## Network Information

```bash
podman inspect --format '{{json .NetworkSettings}}' nginx_container
```

---

# 12. Clean Up Nginx

## Stop

```bash
podman stop nginx_container
```

## Remove

```bash
podman rm nginx_container
```

## Verify

```bash
podman ps -a
```

---

# 13. Useful Podman Commands

| Command                 | Purpose                                  |
| ----------------------- | ---------------------------------------- |
| `podman ps`             | List running containers                  |
| `podman ps -a`          | List all containers                      |
| `podman run`            | Create and start a container             |
| `podman create`         | Create a container without starting it   |
| `podman stop`           | Stop a running container                 |
| `podman restart`        | Restart a container                      |
| `podman rm`             | Remove a container                       |
| `podman rm -f`          | Force-remove a container                 |
| `podman inspect`        | Display detailed container configuration |
| `podman image ls`       | List local images                        |
| `podman pull`           | Pull an image                            |
| `podman system migrate` | Migrate Podman state                     |

---

# ✅ Command Reference Status

All commands documented here were part of the actual Lab 2 execution or verification process.
