# 🛡️ Remediation & Security Recommendations

## Lab 3 — Running Containers with Podman

This document provides security recommendations based on the container-management techniques demonstrated during the lab.

---

## 1. Restrict Published Ports

### Observation

The lab published container port `80` on host ports such as:

```text
8080
8081
8082
```

### Risk

Published ports can make containerized services accessible to other systems depending on the host network configuration.

### Recommendation

Expose only the ports required by the application.

Where appropriate, bind services to a specific interface instead of all interfaces:

```bash
-p 127.0.0.1:8080:80
```

This limits access to the local host.

---

## 2. Use Rootless Containers

### Observation

The lab environment reported:

```text
Rootless=true
```

### Recommendation

Prefer rootless Podman where application requirements allow it.

Rootless execution reduces the privileges available to the container-management process and helps limit the impact of container compromise.

---

## 3. Restrict Bind Mounts

### Observation

The lab mounted:

```text
~/nginx-content
```

into:

```text
/usr/share/nginx/html
```

### Risk

Bind mounts can expose host filesystem content to containers.

### Recommendation

Mount only the directories required by the application.

Avoid unnecessarily mounting sensitive host paths such as:

```text
/etc
/home
/root
/var/run
```

Use read-only mounts when write access is not required:

```bash
-v ~/nginx-content:/usr/share/nginx/html:ro
```

---

## 4. Use Trusted Container Images

### Observation

The lab used:

```text
docker.io/library/nginx:alpine
```

### Recommendation

Use trusted registries and official or verified images.

Where possible:

* Pin images to specific versions.
* Avoid unnecessary third-party images.
* Review image provenance.
* Scan images for vulnerabilities.
* Regularly update images.

Example:

```bash
podman pull docker.io/library/nginx:alpine
```

---

## 5. Avoid Unnecessary Container Privileges

### Recommendation

Containers should not be granted additional privileges unless required.

Avoid unnecessary options such as:

```bash
--privileged
```

Prefer the default restricted security model and explicitly grant only required capabilities.

---

## 6. Apply Resource Limits

### Risk

A container without resource controls may consume excessive CPU or memory.

### Recommendation

For production workloads, consider resource controls such as:

```bash
--memory
--cpus
--pids-limit
```

Example:

```bash
podman run -d \
  --memory=512m \
  --cpus=1 \
  docker.io/library/nginx:alpine
```

---

## 7. Remove Unused Containers

### Observation

The lab performed container cleanup after testing:

```bash
podman rm -f $(podman ps -aq)
```

### Recommendation

Remove unused containers and images to reduce unnecessary attack surface and resource consumption.

Before removing resources, verify that they are not required by another application.

---

## 8. Monitor Container Activity

For production environments, container activity should be monitored.

Useful Podman commands include:

```bash
podman ps
podman stats
podman logs <container>
podman inspect <container>
```

Security monitoring should look for:

* Unexpected processes
* Unexpected network connections
* Repeated container restarts
* Excessive resource usage
* Changes to mounted files
* Unusual container privileges

---

# 🔐 Security Checklist

| Recommendation               | Status                        |
| ---------------------------- | ----------------------------- |
| Use rootless containers      | ✅ Applied                     |
| Restrict published ports     | ⚠️ Review per deployment      |
| Restrict bind mounts         | ⚠️ Review per deployment      |
| Use trusted images           | ✅ Applied                     |
| Avoid unnecessary privileges | ✅ Recommended                 |
| Apply resource limits        | ⚠️ Recommended for production |
| Remove unused containers     | ✅ Applied                     |
| Monitor container activity   | ⚠️ Recommended                |

---

# ✅ Conclusion

The lab used a relatively low-privilege rootless Podman environment and successfully demonstrated basic container operations.

For production deployments, additional controls should be applied around network exposure, filesystem mounts, image security, resource limits, privileges, logging, and monitoring.
