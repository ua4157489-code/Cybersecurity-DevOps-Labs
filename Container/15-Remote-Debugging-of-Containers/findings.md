# Lab 15 — Findings

## Finding 1 — Debug Port Successfully Exposed

The container published debugpy on port `5678`:

```text
5678/tcp -> 0.0.0.0:5678
```

A socket connection test from inside the container returned:

```text
debugpy port: OPEN
```

The host-side test also reported:

```text
Host port 5678: OPEN
```

### Impact

The container provides a reachable debugging interface that can be used by a compatible debugger.

---

## Finding 2 — Application Port Successfully Published

The Flask application port was published as:

```text
3000/tcp -> 0.0.0.0:3000
```

The host-side socket test reported:

```text
Host port 3000: OPEN
```

Because the application was configured with `--wait-for-client`, the HTTP process waits for a debugger connection before serving requests.

---

## Finding 3 — Source Code Successfully Mounted

The host application directory was mounted into:

```text
/app
```

The container contained:

```text
Containerfile
app.py
requirements.txt
```

This confirms that the application source is available directly inside the running container.

---

## Finding 4 — Live Source Updates Work

The source was modified on the host:

```text
Hello from live-mounted debugging container!
```

was changed to:

```text
Hello from updated source code without rebuilding!
```

The updated text was immediately visible inside the running container.

### Result

The source update did not require:

* Image rebuild
* Container recreation

This demonstrates the benefit of bind mounting application source during development and debugging.

---

## Finding 5 — Minimal Image Limitation

The `python:3.12-slim` image did not contain the `ps` command.

Attempting:

```bash
podman exec debug-container ps aux
```

returned:

```text
exec: "ps": executable file not found in $PATH
```

This is consistent with the minimal nature of the base image.

---

## Finding 6 — Debugger Warning

The container logs reported a Python frozen-module warning:

```text
Debugger warning: It seems that frozen modules are being used
```

The same message stated:

```text
Debugging will proceed.
```

Therefore, the message was treated as a debugger warning rather than a container startup failure.

---

## Finding 7 — IDE Configuration Prepared

A VS Code `debugpy` attach configuration was created with:

```text
Port: 5678
Remote root: /app
Local root: app
```

VS Code was not installed in the lab environment, therefore an actual IDE attachment and breakpoint execution were not verified.

## Overall Result

The container-side remote-debugging environment was successfully configured and verified.

The lab demonstrated:

* Debugger port exposure
* Container networking
* Source-code bind mounting
* Live source synchronization
* Debugger configuration
* Container troubleshooting considerations
