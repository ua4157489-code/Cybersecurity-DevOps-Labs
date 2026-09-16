# Lab 15 — Remediation and Security Considerations

## 1. Restrict Debugger Port Exposure

Port `5678` was published on:

```text
0.0.0.0:5678
```

For development environments, exposing a debugger to an untrusted network should be avoided.

Prefer restricting access through:

* Localhost
* SSH tunneling
* Private networks
* Security groups
* Firewall rules

For an EC2 environment, the debug port should generally not be exposed publicly to the Internet.

## 2. Use SSH Tunneling

When debugging remotely from a local development machine, an SSH tunnel can provide a safer path:

```bash
ssh -L 5678:localhost:5678 ubuntu@<EC2-PUBLIC-IP>
```

The tunnel should be created from the developer's local machine rather than from the EC2 instance back to itself.

## 3. Avoid Debugging in Production

Remote debugging interfaces should normally be disabled in production deployments.

Production containers should not use development-oriented debugging configurations unless there is a controlled operational requirement.

## 4. Protect Source Code

The bind mount:

```text
host app/ → /app
```

is useful for development but gives the container access to host-side source files.

Mount only the required directory and avoid unnecessarily mounting sensitive host paths.

## 5. Use Minimal Images Carefully

The `python:3.12-slim` image reduced unnecessary packages but did not contain common troubleshooting utilities such as `ps`.

For production images, minimal images can reduce attack surface.

For troubleshooting environments, temporary diagnostic containers or appropriate debugging tools can be used instead of permanently adding unnecessary packages.

## 6. Use Dependency Pinning

The current requirements file contains:

```text
flask
debugpy
```

For reproducible builds, production-oriented projects should pin dependency versions after validating compatible versions.

Example:

```text
flask==<validated-version>
debugpy==<validated-version>
```

## 7. Protect Debugging Infrastructure

A debugger can provide significant control over the running application process.

Therefore:

* Do not expose debug ports publicly.
* Restrict access to trusted users.
* Use encrypted connections such as SSH.
* Remove debugging configuration from production images.
* Close unused debugging ports.

## 8. Validate Source Changes

Live source mounting is useful during development, but source changes should still be tested before deployment.

A recommended workflow is:

```text
Modify Source
     ↓
Test
     ↓
Build Image
     ↓
Security Scan
     ↓
Deploy
```

## Conclusion

Remote debugging improves development and troubleshooting efficiency, but debugging interfaces increase the attack surface of a containerized application.

The debug port should therefore be treated as a privileged development interface and protected accordingly.
