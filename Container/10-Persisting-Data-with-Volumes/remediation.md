# Lab 10: Remediation and Best Practices

## 1. Use Named Volumes for Application Data

When application data needs to survive container replacement, use a named volume instead of storing important data only inside the container's writable layer.

Example:

```bash
podman run -d \
  --name webapp \
  -v myapp_data:/var/www/html \
  docker.io/library/nginx
```

## 2. Avoid Storing Critical Data Only Inside Containers

Containers are designed to be replaceable.

Important application data should be stored using:

* Named volumes
* Bind mounts where appropriate
* External storage systems for production environments

## 3. Control Bind-Mount Scope

Only mount the directories that the application actually requires.

Avoid unnecessarily exposing large host directories to containers.

Prefer:

```text
~/application-data
```

over exposing an entire home directory.

## 4. Protect Host Data

Bind mounts can expose host files to processes running inside containers.

Use appropriate:

* File permissions
* Directory ownership
* SELinux labeling
* Least-privilege container configurations

The lab used:

```bash
:Z
```

on the bind mount to apply the appropriate SELinux relabeling behavior for the container.

## 5. Back Up Persistent Volumes

Persistent storage should still be backed up.

A named volume surviving container deletion does not protect it from:

* Disk failure
* Accidental deletion
* Host failure
* Data corruption
* Administrative mistakes

## 6. Use Read-Only Mounts When Possible

If a container only needs to read host data, consider using a read-only mount.

Example:

```bash
podman run -d \
  --name readonly_example \
  -v ~/host_data:/usr/share/nginx/html:ro,Z \
  docker.io/library/nginx
```

This reduces the ability of the containerized application to modify host data.

## 7. Remove Unused Volumes

Unused volumes can consume disk space.

Review volumes periodically:

```bash
podman volume ls
```

Inspect individual volumes:

```bash
podman volume inspect <volume_name>
```

Remove volumes only when their data is no longer required:

```bash
podman volume rm <volume_name>
```

## 8. Production Considerations

For production workloads:

* Use persistent storage appropriate to the application.
* Maintain regular backups.
* Apply least-privilege permissions.
* Avoid unnecessary host filesystem exposure.
* Use read-only mounts when write access is unnecessary.
* Monitor storage usage.
* Document ownership and lifecycle of persistent data.

## Conclusion

Persistent storage should be designed separately from the container lifecycle. Named volumes are useful for container-managed application data, while bind mounts are useful when direct host filesystem access is required. Both should be protected with appropriate permissions, access controls, backups, and lifecycle management.
