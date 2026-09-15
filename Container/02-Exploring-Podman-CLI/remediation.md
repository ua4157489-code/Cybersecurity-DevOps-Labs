# Lab 2 — Remediation & Security Recommendations

## 🎯 Purpose

This document provides security and operational recommendations based on the observations made during the Podman CLI lab.

The observed restart timeout and image resolution error are **not treated as confirmed vulnerabilities**. They are documented as configuration and operational observations.

---

# 1. Use Fully Qualified Image References

The short image reference:

```bash
podman run -d --name nginx_container nginx
```

failed because the environment had no unqualified search registries configured.

The successful approach was:

```bash
podman run -d --name nginx_container docker.io/library/nginx
```

### Recommendation

Use fully qualified image references when possible:

```text
docker.io/library/nginx
```

This makes the intended registry explicit and reduces ambiguity.

---

# 2. Use Trusted Image Sources

Container images should come from trusted registries and verified publishers.

For production environments:

* Use approved registries.
* Restrict image sources where appropriate.
* Review image provenance.
* Avoid unknown or untrusted images.
* Maintain an approved image list.

---

# 3. Pin Image Versions

The lab used:

```text
docker.io/library/nginx:latest
```

The `latest` tag can change over time.

### Recommendation

For reproducible deployments, use a specific version:

```text
docker.io/library/nginx:<version>
```

For stronger immutability, use an image digest.

---

# 4. Keep Container Images Updated

Container images should be regularly updated to receive security fixes.

Recommended process:

```text
Pull updated image
        ↓
Scan image
        ↓
Test image
        ↓
Deploy approved image
```

---

# 5. Scan Images for Vulnerabilities

Container images should be scanned before deployment.

Possible tools include:

* Trivy
* Grype
* Enterprise container security scanners

Example workflow:

```text
Image
  ↓
Vulnerability Scanner
  ↓
Review CVEs
  ↓
Remediate
  ↓
Deploy
```

---

# 6. Continue Using Rootless Containers

The lab environment used:

```text
rootless: true
```

### Recommendation

Continue using rootless Podman where application requirements allow it.

Rootless execution reduces the need for privileged host access.

---

# 7. Avoid Privileged Containers

Do not use:

```bash
podman run --privileged ...
```

unless there is a documented technical requirement.

Instead, grant only the permissions required by the application.

---

# 8. Minimize Container Capabilities

The Nginx inspection showed a defined capability set.

For production workloads, capabilities should be minimized.

### Recommendation

Review capabilities using:

```bash
podman inspect <container>
```

Remove unnecessary capabilities where supported by the workload.

---

# 9. Investigate Stop and Restart Timeouts

During the lifecycle test, Podman reported:

```text
StopSignal SIGTERM failed to stop container
in 10 seconds, resorting to SIGKILL
```

### Recommendation

If the behavior occurs repeatedly in a production environment, investigate:

* Container process behavior
* Stop signal
* Stop timeout
* Podman runtime
* User-level service state
* Host configuration
* Container logs
* Podman events

Useful commands include:

```bash
podman ps -a
podman inspect <container>
podman events
```

The observed behavior should be investigated before using the affected workload in production.

---

# 10. Configure Image Registries Carefully

The short-name error occurred because:

```text
no unqualified-search registries are defined
```

### Recommendation

For controlled environments, explicitly configure trusted registries if organizational policy requires short-name resolution.

However, using fully qualified image references is often preferable because the registry is explicit.

---

# 11. Limit Network Exposure

The Nginx container had:

```text
80/tcp
```

inside the container, but no host port was published.

This is a good example of limiting unnecessary external exposure.

### Recommendation

Only publish ports that are actually required.

For example:

```bash
-p 8080:80
```

should only be used when host access to Nginx is required.

---

# 12. Review Container Configuration

Before deploying a container, inspect it:

```bash
podman inspect <container>
```

Review:

* Image
* Image digest
* Command
* Runtime
* Network configuration
* Capabilities
* Privileged status
* Restart policy
* Resource limits
* Mounted volumes
* Environment variables

---

# 13. Apply Resource Limits

Production containers should be protected from excessive resource consumption where appropriate.

Consider configuring:

* CPU limits
* Memory limits
* PID limits
* Storage limits

This helps reduce the impact of resource exhaustion.

---

# 14. Perform Regular Cleanup

Unused containers should be removed.

Check containers:

```bash
podman ps -a
```

Check images:

```bash
podman image ls
```

Regular cleanup reduces unnecessary resources and helps maintain a controlled container environment.

---

# 15. Monitor Container Activity

For production environments, container lifecycle events should be monitored.

Useful Podman functionality includes:

```bash
podman events
```

Monitoring can help identify:

* Unexpected container stops
* Repeated restarts
* Image changes
* Container creation
* Container removal
* Runtime problems

---

# 🔐 Recommended Podman Security Baseline

A practical baseline for future container labs and deployments is:

```text
Rootless execution
        ↓
Trusted image source
        ↓
Fully qualified image reference
        ↓
Pinned image version/digest
        ↓
Image vulnerability scanning
        ↓
Minimal capabilities
        ↓
No unnecessary privileged mode
        ↓
Minimal network exposure
        ↓
Resource limits
        ↓
Runtime monitoring
```

---

# ✅ Remediation Status

| Observation                    | Recommendation                    | Status         |
| ------------------------------ | --------------------------------- | -------------- |
| Unqualified Nginx image failed | Use fully qualified image         | ✅ Applied      |
| Restart timeout observed       | Investigate runtime/stop behavior | ⚠️ Documented  |
| Rootless environment           | Continue rootless execution       | ✅ Recommended  |
| Nginx used `latest`            | Pin version/digest for production | ⚠️ Recommended |
| Container configuration        | Use `podman inspect`              | ✅ Applied      |
| Container cleanup              | Remove unused containers          | ✅ Applied      |
| Image security                 | Scan images                       | ⚠️ Recommended |

---

# 🏁 Final Recommendation

The Lab 2 environment successfully demonstrated Podman container lifecycle management.

For production use, the most important improvements are:

1. Use trusted, fully qualified image references.
2. Pin image versions or digests.
3. Scan images for vulnerabilities.
4. Continue using rootless containers.
5. Avoid unnecessary privileges and capabilities.
6. Limit network exposure.
7. Investigate repeated stop/restart timeouts.
8. Monitor container lifecycle events.
9. Regularly clean unused resources.
10. Inspect and validate container configuration before deployment.
