# Lab 10: Methodology

## 1. Storage Preparation

The lab began by creating a Podman-managed named volume called `myapp_data`.

The volume was listed and inspected to confirm that Podman recognized it as a persistent storage resource.

## 2. Named Volume Mounting

An Nginx container was started with the named volume mounted at:

```text
/var/www/html
```

The container filesystem was checked to confirm that the mount was accessible.

## 3. Data Creation

A test `index.html` file was created inside the mounted directory.

The file contained:

```text
Hello, Volume!
```

The file was read from the container to verify successful data creation.

## 4. Persistence Verification

The original Nginx container was removed.

A new container was then started using the same named volume.

The `index.html` file was read from the new container.

The data remained available, demonstrating that the volume persists independently of the container lifecycle.

## 5. Bind Mount Configuration

A directory named `host_data` was created in the user's home directory.

An `index.html` file was created inside the directory and populated with:

```text
Hello, Bind Mount!
```

The directory was mounted into an Nginx container using a bind mount with the `:Z` option.

## 6. Bind Mount Verification

The file was read from inside the container to verify that the host directory was successfully mapped to the container's web directory.

## 7. Live Update Test

The host file was modified by appending:

```text
Updated content!
```

The file was then checked from both the host and container.

The updated content was visible from inside the container without rebuilding or recreating it.

## 8. Validation

The lab validated two different persistence mechanisms:

```text
Named Volume
     ↓
Persistent storage managed by Podman
     ↓
Survives container removal
```

and:

```text
Bind Mount
     ↓
Host directory
     ↓
Directly reflected inside container
```

## Result

The practical tests confirmed the expected behavior of both named volumes and bind mounts.
