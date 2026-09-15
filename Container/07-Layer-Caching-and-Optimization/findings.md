# Lab 7 — Findings

## Finding 1 — Excessive Layer Creation

### Observation

The initial Dockerfile used separate `RUN` instructions for APT update, package installation, Python installation, and Flask installation.

### Evidence

`podman history myapp:initial` showed separate layers including:

```text
apt-get update                      80 MB
apt-get install curl wget         7.95 MB
apt-get install python3 python3-pip 375 MB
pip install flask                 5.18 MB
```

### Impact

Additional layers can increase image size and make image construction less efficient.

### Severity

**Low / Optimization**

---

## Finding 2 — Initial Image Size

### Observation

The baseline image was approximately:

```text
548 MB
```

### Evidence

```text
548334151 bytes
```

### Impact

Larger images require more storage and may increase transfer and deployment time.

---

## Finding 3 — Layer Consolidation Improvement

### Observation

The optimized Dockerfile combined package installation and cleanup operations.

### Result

The optimized image measured approximately:

```text
467 MB
```

This represents an approximate reduction of:

```text
81 MB
≈ 14.9%
```

compared with the initial image.

---

## Finding 4 — Multi-Stage Build Improvement

### Observation

The multi-stage build separated dependency installation from the runtime image.

### Result

The resulting image was approximately:

```text
85 MB
```

This is approximately **84% smaller** than the 548 MB baseline.

### Impact

A smaller runtime image reduces storage requirements and minimizes unnecessary build tooling in the final container.

---

## Finding 5 — Cache Reuse

### Observation

Podman reported cached build steps during the multi-stage build.

Example:

```text
Using cache
```

### Impact

Unchanged build instructions can be reused, reducing rebuild work.

---

## Finding 6 — Cache Invalidation

### Observation

A changing `CACHEBUST` build argument was used to deliberately invalidate the relevant cache.

### Impact

Cache invalidation is useful when a build step must be forcibly re-executed.

---

## Finding 7 — Runtime Verification

### Observation

The optimized container started successfully.

The application returned:

```text
Hello from optimized container!
```

The Flask logs also recorded:

```text
"GET / HTTP/1.1" 200
```

### Conclusion

The optimization process did not prevent the application from functioning.

---

# Overall Assessment

The lab successfully demonstrated that Dockerfile structure has a direct effect on image layers, caching behavior, and final image size.

The strongest optimization achieved during the lab was the multi-stage image at approximately **85 MB**, compared with the original **548 MB** baseline.
