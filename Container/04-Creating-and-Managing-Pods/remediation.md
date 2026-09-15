# 🛡️ Remediation & Recommendations — Lab 4

## 1. Shared-Volume Container Failure

**Issue**

The additional `nginx2` and `redis2` containers did not remain running.

The following command therefore failed:

```bash
podman exec nginx2 touch /data/testfile
```

Observed error:

```
Error: can only create exec sessions on running containers: container state improper
```

**Recommended Investigation**

Check all container states:

```bash
podman ps -a --pod
```

Inspect logs:

```bash
podman logs nginx2
podman logs redis2
```

Inspect the container state:

```bash
podman inspect nginx2 --format '{{.State.Status}}'
podman inspect redis2 --format '{{.State.Status}}'
```

The root cause should be identified before repeating the file-sharing test.

## 2. Validate Container Startup Behavior

Before using a container for a shared-volume test, confirm that the container's main process remains active.

```bash
podman ps -a
```

If a container exits immediately, inspect its logs and configuration rather than assuming the volume configuration is the cause.

## 3. Use Appropriate Volume Permissions

When a container only needs to read shared data, consider a read-only mount:

```bash
-v shared-vol:/data:ro
```

Read-only mounts reduce the risk of accidental modification.

Use writable mounts only where write access is actually required.

## 4. Restrict Host Port Exposure

The lab exposed:

```
8080:80
```

For services that should only be reachable locally, consider binding the host port to loopback:

```bash
-p 127.0.0.1:8080:80
```

This reduces unnecessary network exposure.

## 5. Use Trusted Images

For production deployments:

- Use trusted image registries.
- Review image provenance.
- Prefer pinned image versions or digests.
- Regularly scan images for vulnerabilities.
- Avoid using outdated images.

For example, production deployments can pin images by digest rather than relying only on a mutable tag such as `alpine`.

## 6. Avoid Unnecessary Privileges

Continue using rootless Podman where practical.

Avoid:

```
--privileged
```

unless the workload explicitly requires it.

Apply the principle of least privilege to containers, volumes, and host resources.

## 7. Monitor Pod Health

A pod entering a `Degraded` state should be investigated.

Useful commands:

```bash
podman pod ps
podman ps -a --pod
podman pod inspect demo-pod
```

Container failures should be investigated before treating the deployment as healthy.

## 8. Add Health Checks Where Appropriate

For production workloads, configure health checks so service availability can be monitored independently of container process state.

This helps detect services that are running but not actually responding.

## 9. Retest the Shared Volume

After correcting the container startup issue, repeat:

```bash
podman exec nginx2 touch /data/testfile
```

Then verify from the second container:

```bash
podman exec redis2 ls -la /data
```

The expected successful result would be the presence of:

```
testfile
```

This should only be documented after the result is actually observed.

## 10. Maintain Evidence-Based Documentation

For future labs:

- Run the command.
- Verify the output.
- Capture the screenshot.
- Record failures exactly.
- Apply remediation.
- Retest.
- Document only verified results.

This keeps the GitHub portfolio technically accurate and reproducible.

## 11. Cleanup

After testing:

```bash
podman pod rm -f demo-pod
podman volume rm shared-vol
```

Verify:

```bash
podman pod ps
podman ps -a
podman volume ls
```

## 12. Final Recommendation

The core Podman pod deployment was successful. The main remediation priority is investigating why the additional volume-mounted containers exited. Once their startup behavior is understood and corrected, the shared-volume test should be repeated and documented with actual evidence.
