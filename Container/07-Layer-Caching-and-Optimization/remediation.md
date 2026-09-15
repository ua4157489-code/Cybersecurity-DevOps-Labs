# Lab 7 — Remediation and Optimization Recommendations

## 1. Consolidate Related RUN Instructions

Instead of:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl wget
RUN apt-get install -y python3 python3-pip
RUN pip install flask
```

combine related operations where appropriate:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl wget python3 python3-pip && \
    pip install flask && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Benefit

Reduces unnecessary layers and removes package metadata from the resulting layer.

---

## 2. Order Dockerfile Instructions for Effective Caching

Place relatively stable instructions before frequently changing application files.

For example:

```dockerfile
RUN ...install dependencies...
COPY app.py /app/
```

### Benefit

Changing `app.py` does not require dependency installation to be repeated when the dependency layer remains unchanged.

---

## 3. Use Multi-Stage Builds

Separate build dependencies from the runtime image.

### Benefit

The final image can exclude unnecessary build tools and intermediate files.

This lab reduced the image from approximately:

```text
548 MB
```

to:

```text
85 MB
```

using the multi-stage approach.

---

## 4. Clean Package Manager Metadata

For Ubuntu-based images:

```bash
apt-get clean
rm -rf /var/lib/apt/lists/*
```

### Benefit

Removes package metadata that is not required at runtime.

---

## 5. Use a Dependency File

Application dependencies should be maintained in a dedicated file such as:

```text
requirements.txt
```

Example:

```text
Flask
```

### Benefit

Makes dependency management clearer and improves build reproducibility.

For production projects, dependencies should preferably be version-pinned.

---

## 6. Use Cache Busting Carefully

A build argument can intentionally invalidate a cache:

```dockerfile
ARG CACHEBUST=1
```

and:

```bash
--build-arg CACHEBUST=$(date +%s)
```

### Recommendation

Use cache busting only when necessary. Unnecessary cache invalidation can increase build time.

---

## 7. Use Production-Grade Application Servers

The lab uses Flask's built-in development server for demonstration.

For production deployments, use a production WSGI server such as Gunicorn or another appropriate server.

---

## 8. Keep Base Images Updated

Regularly rebuild images using maintained base images and apply security updates.

### Benefit

Reduces exposure to known vulnerabilities in outdated operating-system packages.

---

## 9. Scan Container Images

Integrate image vulnerability scanning into the CI/CD pipeline.

Recommended checks include:

* Known CVEs
* Outdated packages
* Misconfigurations
* Excessive privileges
* Secrets accidentally included in image layers

---

## 10. Avoid Secrets in Dockerfiles

Do not place passwords, API keys, private keys, or tokens directly inside:

```dockerfile
ENV
RUN
COPY
```

instructions.

Use an appropriate secret-management mechanism instead.

---

# Final Recommendation

For production container builds, prefer:

1. Small and maintained base images.
2. Multi-stage builds where appropriate.
3. Consolidated package operations.
4. Effective layer ordering.
5. Dependency pinning.
6. Removal of temporary package metadata.
7. Regular vulnerability scanning.
8. Production-grade application servers.
9. Non-root execution where practical.
10. Minimal runtime dependencies.

These practices improve image size, build performance, security, and maintainability.
