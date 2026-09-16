# Lab 15 — Methodology

## Objective

The objective of this lab was to demonstrate how a containerized Python application can expose a remote debugging interface while allowing its source code to be modified without rebuilding the container image.

## Methodology

### Phase 1 — Application Preparation

A lightweight Python Flask application was created with two HTTP endpoints:

* `/`
* `/status`

The application dependencies were defined in `requirements.txt`:

```text
flask
debugpy
```

### Phase 2 — Container Image

A `python:3.12-slim` base image was used.

The application dependencies were installed during image creation, and ports `3000` and `5678` were exposed.

`debugpy` was configured to listen on:

```text
0.0.0.0:5678
```

The application was configured to wait for a debugger connection using:

```text
--wait-for-client
```

### Phase 3 — Container Deployment

The image was launched as:

```text
debug-container
```

with:

```text
3000 → Flask application
5678 → debugpy
```

Both ports were published through Podman.

### Phase 4 — Debugger Verification

The debugpy socket was tested from inside the container using Python's socket module.

The result was:

```text
debugpy port: OPEN
```

The host-side test also confirmed:

```text
Host port 3000: OPEN
Host port 5678: OPEN
```

### Phase 5 — Source Code Mount

The host application directory was bind mounted to:

```text
/app
```

inside the container.

The mounted directory was verified using:

```bash
podman exec debug-container ls -la /app
```

### Phase 6 — Live Source Modification

The Flask response was changed directly on the host.

The updated source was then read from inside the running container.

The modified source was visible immediately, demonstrating that the bind mount synchronized the host and container filesystems.

No image rebuild was performed.

### Phase 7 — Debugger Configuration

A VS Code `debugpy` attach configuration was created with:

```text
host: localhost
port: 5678
```

and the following path mapping:

```text
localRoot  → app
remoteRoot → /app
```

VS Code itself was not installed in the lab environment, so an actual debugger attachment and breakpoint test were not performed.

## Expected Debugging Workflow

When a compatible debugger connects to port `5678`, the process configured with `--wait-for-client` can continue execution and start the Flask application.

The intended workflow is:

```text
Source Code
    │
    ▼
Bind Mount
    │
    ▼
Container /app
    │
    ▼
Python + Flask
    │
    ├── :3000 → Application
    │
    └── :5678 → debugpy
                    │
                    ▼
                 Debugger
```

## Verification Strategy

The following evidence was collected:

1. Image creation
2. Container status
3. Published ports
4. Debugpy socket availability
5. Mounted source files
6. Live source-code synchronization
7. Debugger configuration

This provided evidence for the container-side remote-debugging workflow without claiming an unperformed IDE debugging session.
