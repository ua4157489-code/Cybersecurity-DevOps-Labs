# Lab 7 — Methodology

## 1. Baseline Construction

A baseline Ubuntu 22.04 image was created using multiple independent `RUN` instructions.

This intentionally demonstrated how each filesystem-changing instruction can create an additional image layer.

## 2. Baseline Measurement

The baseline image was inspected using Podman image and history commands.

The measured size was:

```text
548334151 bytes
≈ 548 MB
```

Layer history showed large layers associated with APT operations and Python installation.

## 3. Layer Optimization

The separate package installation commands were consolidated into a single `RUN` instruction.

APT metadata was removed after installation:

```bash
apt-get clean
rm -rf /var/lib/apt/lists/*
```

This reduced unnecessary data retained in the resulting image layer.

## 4. Cache Testing

The application source was modified without changing dependency installation instructions.

The image was rebuilt to observe Podman's ability to reuse previously generated layers.

This demonstrated why Dockerfiles should generally place relatively stable dependency installation steps before frequently changing application source files.

## 5. Cache Invalidation

A build argument was introduced:

```dockerfile
ARG CACHEBUST=1
```

A timestamp was passed during the build:

```bash
--build-arg CACHEBUST=$(date +%s)
```

This provided a controlled method of invalidating the relevant cache.

## 6. Image Inspection

Podman history and inspect commands were used to analyze:

* Image size
* Layer structure
* Commands used to create layers
* Runtime configuration
* Root filesystem information
* Image metadata

## 7. Multi-Stage Build

A builder stage installed Python and Flask dependencies.

A separate runtime stage copied only the required Python packages and application files.

This prevented the builder-stage installation layers from becoming part of the final runtime image.

## 8. Runtime Validation

The optimized image was started as a container and exposed on port 8080.

The application was then tested with `curl`.

Container logs were also examined to verify successful Flask startup and HTTP request handling.

## 9. Evidence Collection

Screenshots were captured for:

* Dockerfiles and application
* Initial image
* Image history
* Image inspection
* Cache behavior
* Cache busting
* Multi-stage build
* Running container
* Application response
* Container logs

## 10. Success Criteria

The lab was considered successful when:

* Initial image was successfully built.
* Optimized image was smaller than the initial image.
* Cache behavior was demonstrated.
* Cache invalidation was demonstrated.
* Multi-stage image was successfully built.
* Runtime container started successfully.
* Flask application returned the expected response.
* HTTP request returned status `200`.
