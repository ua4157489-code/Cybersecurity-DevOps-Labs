# Lab 2 — Methodology

## 🎯 Purpose

The purpose of this lab was to develop practical familiarity with the **Podman command-line interface** and understand the lifecycle and configuration of Linux containers.

The methodology focused on performing each task directly on the test host, validating the result, recording observations, and cleaning up test resources.

---

# 1. Scope

The lab covered:

* Podman environment verification
* Container listing
* Container creation
* Container execution
* Container stopping
* Container restarting
* Container removal
* Image resolution
* Nginx deployment
* Container inspection
* Cleanup

---

# 2. Test Environment

The practical work was performed on:

```text
Operating System: Ubuntu 24.04.3 LTS
Podman Version: 4.9.3
Architecture: AMD64
User: ubuntu
Execution Mode: Rootless
Runtime: runc
Network: slirp4netns
Storage: overlay
Environment: AWS EC2
```

---

# 3. Methodology Phase 1 — Environment Verification

The first step was to verify the host and Podman installation.

Commands:

```bash
cat /etc/os-release
podman --version
```

This established the operating system and container engine version before testing.

---

# 4. Methodology Phase 2 — Container Inventory

The existing container state was checked using:

```bash
podman ps
podman ps -a
```

This ensured that the lab started from a known state.

---

# 5. Methodology Phase 3 — Alpine Container Test

An Alpine Linux container was created:

```bash
podman run -it --name my_alpine alpine sh
```

Inside the container, the following information was collected:

```bash
cat /etc/os-release
hostname
```

The container was then exited using:

```bash
exit
```

The host was used to verify the resulting container state.

---

# 6. Methodology Phase 4 — Lifecycle Testing

The container lifecycle was tested using:

```bash
podman stop
podman restart
podman rm
```

The first restart test used the interactive Alpine container.

Because its primary `sh` process had exited, it did not remain running after restart.

A second lifecycle test used:

```bash
podman run -d --name my_alpine_lifecycle alpine sleep 300
```

This allowed restart behavior to be tested with a longer-running process.

The observed stop timeout was recorded instead of being omitted.

---

# 7. Methodology Phase 5 — Container Removal

A container was created without starting it:

```bash
podman create --name my_alpine alpine sh
```

It was then removed:

```bash
podman rm my_alpine
```

The container inventory was checked afterward to confirm successful cleanup.

---

# 8. Methodology Phase 6 — Nginx Deployment

Nginx deployment was first attempted using:

```bash
podman run -d --name nginx_container nginx
```

The command failed because the environment had no configured unqualified image search registry.

Instead of changing the system configuration unnecessarily, a fully qualified trusted image reference was used:

```bash
podman run -d --name nginx_container docker.io/library/nginx
```

The container then started successfully.

---

# 9. Methodology Phase 7 — Container Inspection

The running Nginx container was inspected using:

```bash
podman inspect nginx_container
```

Formatted inspection commands were then used to extract specific values:

```bash
podman inspect --format '{{.Id}}' nginx_container
podman inspect --format '{{.State.Status}}' nginx_container
podman inspect --format '{{.Config.Image}}' nginx_container
podman inspect --format '{{.Config.Cmd}}' nginx_container
```

This provided focused evidence for the container's identity, state, image, and process configuration.

---

# 10. Methodology Phase 8 — Evidence Collection

The lab screenshots were captured from the actual terminal session.

The final evidence set contains:

```text
screenshots/
├── 01-podman-environment.png
├── 02-list-containers.png
├── 03-alpine-container.png
├── 04-nginx-inspect.png
└── 05-nginx-cleanup.png
```

No screenshot was created for the restart timeout or short-name resolution error, so those events are documented as terminal observations rather than screenshot evidence.

---

# 11. Methodology Phase 9 — Cleanup

The Nginx container was stopped and removed:

```bash
podman stop nginx_container
podman rm nginx_container
```

The final state was verified:

```bash
podman ps -a
```

The final container inventory was empty.

---

# 12. Evidence Validation

The documentation was prepared from actual commands and observed results.

The following principles were followed:

* No fabricated output
* No fabricated screenshots
* Actual container IDs were retained where relevant
* Environment-specific errors were documented
* Successful and unsuccessful tests were both recorded
* Cleanup was verified

---

# ✅ Conclusion

The methodology successfully covered the core Podman CLI lifecycle.

The lab also demonstrated why actual runtime behavior should be validated in the target environment. The interactive Alpine container behaved differently from a long-running process, while the Nginx deployment required a fully qualified image reference because of the host's registry configuration.
