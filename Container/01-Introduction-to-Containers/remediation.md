# Remediation & Security Recommendations

## Overview

This lab did not identify a security vulnerability. The purpose was to understand basic containerization and isolation.

However, container environments should still be securely configured before being used in production.

---

## 1. Prefer Rootless Containers

Run containers as a non-root user whenever possible.

Example:

```bash
podman run hello-world
```

Rootless Podman reduces the impact of a compromised container because the container process does not require host-level root privileges.

---

## 2. Use Trusted Images

Only use container images from trusted and verified sources.

Example:

```bash
podman pull docker.io/library/alpine
```

Organizations should establish approved container registries and image sources.

---

## 3. Keep Images Updated

Container images should be regularly updated to receive security fixes.

Administrators should periodically rebuild containers using current base images.

---

## 4. Minimize Container Privileges

Avoid unnecessary privileges and capabilities.

Containers should only receive the permissions required for their workload.

---

## 5. Avoid Running Unnecessary Services

A container should contain only the components required by its application.

Reducing unnecessary packages and services reduces the attack surface.

---

## 6. Scan Container Images

Before deploying containers into production, images should be scanned for known vulnerabilities.

Common tools include:

* Trivy
* Grype
* Clair

---

## 7. Use Resource Controls

CPU and memory limits should be applied to production workloads where appropriate.

This can help reduce the impact of resource exhaustion and denial-of-service conditions.

---

## 8. Monitor Container Activity

Production container environments should be monitored for:

* Unexpected processes
* Suspicious network connections
* Privilege escalation attempts
* Image changes
* Container creation
* Container deletion

---

## 9. Secure Container Registries

Private registries should use:

* Authentication
* TLS
* Access controls
* Image signing
* Vulnerability scanning

---

## Conclusion

No remediation was required for the exercises performed in this lab.

The recommendations above establish basic security practices that should be considered when moving from a learning environment to production container infrastructure.
