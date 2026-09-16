# Lab 13: Troubleshooting Containers — Remediation

## Purpose

Although the Nginx container used in this lab was functioning correctly, the following remediation practices can be applied when troubleshooting real container failures.

## 1. Investigate Logs First

Use container logs to identify startup failures, application errors, and HTTP failures.

```bash
podman logs <container>
podman logs --since 5m <container>
podman logs --tail 50 <container>
```

Look for:

* Error messages.
* Failed configuration.
* Permission errors.
* Connection failures.
* Repeated application crashes.

## 2. Inspect Container State

Check whether the container is running, stopped, restarting, or has exited.

```bash
podman inspect <container>
```

Pay particular attention to:

```text
State.Status
State.ExitCode
State.Error
State.OOMKilled
RestartCount
```

## 3. Check Resource Consumption

Use:

```bash
podman stats --no-stream <container>
```

If memory or CPU consumption is excessive:

* Investigate the application workload.
* Review memory leaks.
* Apply appropriate resource limits.
* Check for runaway processes.

## 4. Debug from Inside the Container

Use:

```bash
podman exec -it <container> /bin/sh
```

Then inspect:

```bash
ps aux
```

and relevant application configuration files.

## 5. Validate Configuration

For Nginx:

```bash
nginx -t
```

For other applications, use the application's native configuration validation command where available.

Configuration should be validated before restarting production services.

## 6. Verify Internal Connectivity

Test the application from inside the container:

```bash
wget -qO- http://localhost
```

or, if available:

```bash
curl http://localhost
```

This helps distinguish application problems from external networking problems.

## 7. Verify Port Publishing

Check published ports:

```bash
podman port <container>
```

Then test from the host:

```bash
curl -I http://localhost:<port>
```

## 8. Review Network Configuration

Inspect the container network configuration:

```bash
podman inspect <container>
```

Pay attention to the configured network mode and port mappings.

Rootless Podman commonly uses user-space networking such as `slirp4netns`, so the container's inspected IP address may not behave like a traditional bridge-network IP.

## 9. Restart a Faulty Container

If configuration and logs indicate that restarting is appropriate:

```bash
podman restart <container>
```

If the container remains unhealthy, investigate the underlying cause instead of repeatedly restarting it.

## 10. Recreate Misconfigured Containers

When configuration changes cannot be safely applied to an existing container, recreate it with the correct parameters.

Before removal, verify whether the container contains important persistent data.

## 11. Security Recommendations

For production deployments:

* Run containers with the minimum required privileges.
* Avoid unnecessary Linux capabilities.
* Avoid exposing unnecessary ports.
* Keep images patched and updated.
* Monitor container logs.
* Apply CPU and memory limits where appropriate.
* Use persistent storage for stateful application data.
* Avoid embedding secrets directly into images.
* Regularly remove unused images and containers.

## Final Remediation Result

No remediation was required for the Nginx container used in this lab because:

* The container was running.
* Exit code was `0`.
* No runtime error was reported.
* Nginx configuration passed validation.
* Internal connectivity succeeded.
* External HTTP connectivity returned `200 OK`.
* Resource consumption remained low.

The lab therefore demonstrated the troubleshooting process on a healthy container while establishing commands and procedures applicable to failed or misconfigured containers.
