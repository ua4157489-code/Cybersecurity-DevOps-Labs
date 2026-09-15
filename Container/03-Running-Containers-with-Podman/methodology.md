# 🧪 Methodology

## Lab 3 — Running Containers with Podman

## 1. Objective

The objective of this lab was to gain practical experience with Podman container operations by deploying an Nginx container and testing:

* Detached container execution
* Host-to-container port mapping
* Host directory mounting
* Custom container naming
* Container inspection
* Container cleanup

---

## 2. Environment Preparation

The Podman environment was verified before beginning the practical exercises.

```bash
podman --version
```

The runtime configuration was checked using:

```bash
podman info --format 'Rootless={{.Host.Security.Rootless}} | Runtime={{.Host.OCIRuntime.Name}} | Network={{.Host.NetworkBackend}}'
```

The environment used:

```text
Rootless=true
Runtime=runc
Network=netavark
```

---

## 3. Container Image Preparation

The Nginx Alpine image was obtained from the Docker Hub library:

```bash
podman pull docker.io/library/nginx:alpine
```

The local image list was then verified:

```bash
podman images
```

---

## 4. Detached Container Testing

A basic Nginx container was launched using detached mode:

```bash
podman run -d docker.io/library/nginx:alpine
```

The running container was verified:

```bash
podman ps
```

This confirmed that the container could run independently of the interactive terminal.

---

## 5. Port Mapping Validation

Existing containers were stopped before starting the port-mapping test.

```bash
podman stop $(podman ps -q)
```

Nginx was then started with host port `8080` mapped to container port `80`:

```bash
podman run -d -p 8080:80 docker.io/library/nginx:alpine
```

The mapping was verified:

```bash
podman port <container_id>
```

Application-level connectivity was tested:

```bash
curl http://localhost:8080
```

The returned Nginx welcome page confirmed that the port mapping was functional.

---

## 6. Volume Mount Validation

A host directory was created:

```bash
mkdir -p ~/nginx-content
```

Test content was written to the host:

```bash
echo "Hello from host!" > ~/nginx-content/index.html
```

The directory was mounted into the Nginx document root:

```bash
podman run -d -p 8081:80 \
  -v ~/nginx-content:/usr/share/nginx/html:Z \
  docker.io/library/nginx:alpine
```

The mounted content was tested:

```bash
curl http://localhost:8081
```

The response:

```text
Hello from host!
```

confirmed that Nginx was serving the host-mounted file.

The mount configuration was additionally inspected with:

```bash
podman inspect <container_name> --format '{{json .Mounts}}'
```

---

## 7. Custom Naming Validation

A new container was created with a custom name:

```bash
podman run -d --name my-nginx -p 8082:80 docker.io/library/nginx:alpine
```

The container state was verified:

```bash
podman inspect my-nginx --format '{{.State.Status}}'
```

The port mapping was checked:

```bash
podman port my-nginx
```

The application was tested:

```bash
curl http://localhost:8082
```

---

## 8. Cleanup Procedure

After all tests were completed, the running containers were stopped:

```bash
podman stop $(podman ps -q)
```

All containers were then removed:

```bash
podman rm -f $(podman ps -aq)
```

The final container state was verified:

```bash
podman ps -a
```

The Nginx image was checked:

```bash
podman images
```

Finally, the host-mounted content was verified:

```bash
cat ~/nginx-content/index.html
```

This confirmed that the host data persisted after container removal.

---

## 9. Evidence Collection

Practical evidence is stored under:

```text
screenshots/
```

The planned evidence files are:

```text
01-podman-environment.png
02-detached-container.png
03-port-mapping.png
04-volume-mount.png
05-custom-name.png
06-cleanup.png
```

These screenshots document the environment, execution, networking, storage, naming, and cleanup stages of the lab.

---

# ✅ Methodology Result

Each objective was validated through direct command execution and service-level testing.

The methodology confirms successful completion of:

* Container execution
* Port publishing
* Volume mounting
* Container naming
* Container inspection
* Resource cleanup
