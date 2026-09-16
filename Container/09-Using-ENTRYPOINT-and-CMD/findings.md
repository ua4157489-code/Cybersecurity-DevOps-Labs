# Lab 9: Using ENTRYPOINT and CMD — Findings

## Finding 1: ENTRYPOINT Defines the Main Execution Context

The basic image used:

```dockerfile
ENTRYPOINT ["echo", "Entrypoint says:"]
CMD ["Default CMD message"]
```

Running the image produced:

```text
Entrypoint says: Default CMD message
```

This demonstrated that the `CMD` value was passed as an argument to the configured `ENTRYPOINT`.

---

## Finding 2: CMD Can Be Replaced at Runtime

The default command:

```text
Default CMD message
```

was replaced by:

```text
Custom message
```

using:

```bash
podman run --rm entrypoint-demo "Custom message"
```

The observed result was:

```text
Entrypoint says: Custom message
```

The `ENTRYPOINT` remained unchanged while the `CMD` argument was replaced.

---

## Finding 3: ENTRYPOINT Can Be Implemented Using a Script

The lab successfully used:

```text
/usr/local/bin/greet.sh
```

as the container entrypoint.

The script accepted two positional arguments:

```text
$1
$2
```

The runtime test:

```bash
podman run --rm greet-demo "Container Workshop" "Instructor"
```

produced:

```text
Welcome to Container Workshop from Instructor
```

This confirmed that runtime arguments were passed to the entrypoint script.

---

## Finding 4: ENTRYPOINT Can Be Replaced

The configured script entrypoint was replaced with:

```bash
--entrypoint echo
```

The resulting output was:

```text
This completely replaces the ENTRYPOINT
```

The original `greet.sh` script therefore did not execute during this test.

---

## Finding 5: Shell Form Behaves Differently

The shell-form configuration was:

```dockerfile
ENTRYPOINT echo "Shell form ENTRYPOINT:"
CMD echo "Shell form CMD"
```

The observed output was:

```text
Shell form ENTRYPOINT:
```

This demonstrated that shell-form instructions do not behave identically to the exec-form `ENTRYPOINT` and `CMD` combination tested earlier.

---

## Finding 6: Exec Form Provides Explicit Process Configuration

The exec-form syntax:

```dockerfile
ENTRYPOINT ["echo", "Entrypoint says:"]
```

explicitly defines the executable and its arguments.

This makes exec form useful when predictable process execution and argument handling are required.

---

## Summary

The practical tests demonstrated:

* `ENTRYPOINT` defines the primary container execution behavior.
* `CMD` provides default arguments or command behavior.
* Runtime arguments can replace the default `CMD`.
* `--entrypoint` can replace the configured entrypoint.
* Scripts can be used as container entrypoints.
* Shell-form and exec-form instructions have different execution behavior.
