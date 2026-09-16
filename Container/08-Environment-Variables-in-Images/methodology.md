# Lab 8: Methodology

## 1. Environment Preparation

A dedicated working directory was created for the environment-variable lab. Podman was used as the container engine.

## 2. Image Configuration

A UBI 8 minimal base image was selected:

```text
registry.access.redhat.com/ubi8/ubi-minimal
```

The initial Containerfile defined three application variables using `ENV`:

```text
APP_NAME
APP_VERSION
APP_ENV
```

## 3. Image Build

The image was built with:

```bash
podman build -t env-demo .
```

This created an image containing the configured environment variables.

## 4. Default Runtime Test

The image was started without additional environment arguments.

This verified that the variables defined using `ENV` were available to the application at runtime.

## 5. Runtime Override Test

The `-e` option was used to override selected variables during container startup.

This demonstrated that runtime configuration can take precedence over the default values defined in the image.

## 6. Environment File Test

An external `app.env` file was created containing application configuration.

Podman's `--env-file` option was then used to load the variables into the container.

This demonstrates a practical method for separating configuration from the image.

## 7. Environment Inspection

A long-running container was started to allow inspection with `podman exec`.

The environment was examined directly inside the running container and through `podman inspect`.

## 8. Build-Time ARG Test

The Containerfile was modified to use:

```dockerfile
ARG APP_BUILD_NUMBER
```

The argument was passed during the image build using:

```bash
--build-arg APP_BUILD_NUMBER=42
```

The value was assigned to an `ENV` variable so it could be accessed when the container started.

## 9. Verification

The resulting image was executed and the build number was displayed.

This confirmed the complete flow:

```text
Build argument
      ↓
ARG
      ↓
ENV
      ↓
Running container
```

## 10. Evidence Collection

Screenshots were captured for:

* Containerfile configuration
* Image build
* Default environment
* Runtime override
* Environment file
* Environment inspection
* ARG build
* ARG runtime verification

## Success Criteria

The lab was considered successful when:

* `ENV` variables were available at runtime.
* Runtime overrides worked.
* An environment file could configure the container.
* Environment variables could be inspected.
* `ARG` could be passed during image construction.
* The resulting build value could be accessed at runtime.
