# 🧪 Methodology — Lab 4

## 1. Scope

This lab evaluates basic Podman pod management and focuses on:

- Pod creation
- Multi-container deployment
- Port mapping
- Shared networking
- Volume creation
- Container-state validation
- Troubleshooting
- Cleanup

The assessment was performed on an Ubuntu AWS EC2 host using rootless Podman.

## 2. Environment Validation

The Podman installation was verified before beginning the deployment.

```bash
podman --version
podman info --format 'Rootless={{.Host.Security.Rootless}} | Runtime={{.Host.OCIRuntime.Name}} | Network={{.Host.NetworkBackend}}'
```

This established the runtime, rootless mode, and network backend used during testing.

## 3. Image Preparation

The required Nginx and Redis images were pulled from Docker Hub:

```bash
podman pull docker.io/library/nginx:alpine
podman pull docker.io/library/redis:alpine
```

Only the required images were used for the exercise.

## 4. Pod Deployment

A dedicated pod named `demo-pod` was created with host port `8080` mapped to container port `80`.

```bash
podman pod create --name demo-pod -p 8080:80
```

The pod was then verified before additional containers were deployed.

## 5. Container Deployment

Nginx was deployed first:

```bash
podman run -d --pod demo-pod --name nginx-container docker.io/library/nginx:alpine
```

Redis was then deployed into the same pod:

```bash
podman run -d --pod demo-pod --name redis-container docker.io/library/redis:alpine
```

Container state was checked with:

```bash
podman ps --pod
```

## 6. Service Validation

Nginx was tested externally:

```bash
curl http://localhost:8080
```

Redis was tested internally:

```bash
podman exec redis-container redis-cli ping
```

A successful `PONG` response confirmed Redis availability.

## 7. Network Validation

The pod's shared network namespace was validated by requesting the Nginx service through localhost from inside the container:

```bash
podman exec nginx-container sh -c 'wget -qO- http://127.0.0.1:80 | head'
```

This provided practical evidence of shared pod networking.

## 8. Volume Validation

A named volume was created:

```bash
podman volume create shared-vol
podman volume inspect shared-vol
```

The volume was then mounted into additional containers.

The containers did not remain running, so the file-sharing test was not treated as successful.

## 9. Failure Handling

When the additional containers were found to be unavailable, container state was checked rather than assuming successful execution.

Recommended diagnostic commands:

```bash
podman ps -a --pod
podman logs nginx2
podman logs redis2
podman inspect nginx2 --format '{{.State.Status}}'
podman inspect redis2 --format '{{.State.Status}}'
```

The observed `container state improper` error was recorded as evidence.

## 10. Evidence Standard

Only results actually observed during the lab are documented.

Successful evidence includes:

- Pod creation
- Nginx response
- Redis PONG
- Shared networking test
- Volume creation and inspection
- Cleanup

The shared-volume file test is explicitly documented as incomplete.

## 11. Cleanup

All created resources were removed after testing:

```bash
podman pod rm -f demo-pod
podman volume rm shared-vol
```

Final verification:

```bash
podman pod ps
podman ps -a
podman volume ls
```

## 12. Methodology Conclusion

The methodology combined deployment, validation, state inspection, failure handling, evidence collection, and cleanup. This approach ensures that the documentation reflects actual observed behavior rather than expected behavior alone.
