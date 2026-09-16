# Lab 8: Environment Variables in Images

## Overview

This lab demonstrates how environment variables and build-time arguments can be used to configure containerized applications with Podman.

The lab covers:

* Defining environment variables with `ENV`
* Overriding variables at runtime with `-e`
* Loading variables from an environment file
* Inspecting environment variables inside a running container
* Using `ARG` for build-time configuration
* Passing build arguments with `--build-arg`

## Objectives

By completing this lab, the following container configuration concepts were practiced:

* Use `ENV` and `ARG` in a Containerfile
* Define persistent environment variables in an image
* Override environment variables at runtime
* Use environment files for container configuration
* Inspect environment variables in running containers
* Use build-time arguments to create configurable images

## Environment

| Component         | Details                                       |
| ----------------- | --------------------------------------------- |
| Container Engine  | Podman                                        |
| Base Image        | `registry.access.redhat.com/ubi8/ubi-minimal` |
| Host OS           | Ubuntu Linux                                  |
| Architecture      | x86_64 / amd64                                |
| Application Image | `env-demo`                                    |
| ARG Image         | `arg-demo`                                    |

## Project Structure

```text
08-Environment-Variables-in-Images/
├── Containerfile
├── app.env
├── README.md
├── command.md
├── methodology.md
├── findings.md
├── remediation.md
└── screenshots/
    ├── 01-containerfile-env.png
    ├── 02-env-build.png
    ├── 03-default-env.png
    ├── 04-runtime-env-override.png
    ├── 05-env-file-override.png
    ├── 06-environment-inspection.png
    ├── 07-arg-build.png
    └── 08-arg-runtime.png
```

---

# Task 1: Define Environment Variables in Containerfile

## 1.1 Containerfile

The initial Containerfile uses the `ENV` instruction to define application configuration:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal

ENV APP_NAME="MyApp" \
    APP_VERSION="1.0" \
    APP_ENV="development"

CMD echo "Running $APP_NAME v$APP_VERSION in $APP_ENV mode"
```

The three variables are:

* `APP_NAME` — application name
* `APP_VERSION` — application version
* `APP_ENV` — application environment

### Screenshot 1 — Containerfile with ENV

![Containerfile with environment variables](screenshots/01-containerfile-env.png)

**Explanation:**
This screenshot shows the Containerfile containing the `ENV` instruction and the three application environment variables. It demonstrates how default runtime configuration can be defined directly in a container image.

---

## 1.2 Build the Image

The image was built using:

```bash
podman build -t env-demo .
```

### Screenshot 2 — Building the Environment Image

![Environment image build](screenshots/02-env-build.png)

**Explanation:**
This screenshot shows the Podman build process for the `env-demo` image. A successful build confirms that the Containerfile and its environment-variable configuration were accepted by the container engine.

---

## 1.3 Run with Default Variables

The image was executed without providing any runtime overrides:

```bash
podman run --rm env-demo
```

Output:

```text
Running MyApp v1.0 in development mode
```

### Screenshot 3 — Default Environment Variables

![Default environment variables](screenshots/03-default-env.png)

**Explanation:**
This screenshot verifies that the container automatically uses the values defined by `ENV` in the image:

```text
APP_NAME=MyApp
APP_VERSION=1.0
APP_ENV=development
```

No runtime overrides were required.

---

# Task 2: Override Environment Variables at Runtime

## 2.1 Override Using Command Line

The `-e` option was used to override the image defaults:

```bash
podman run --rm \
  -e APP_ENV="production" \
  -e APP_VERSION="2.0" \
  env-demo
```

The expected configuration becomes:

```text
APP_NAME=MyApp
APP_VERSION=2.0
APP_ENV=production
```

### Screenshot 4 — Runtime Environment Override

![Runtime environment override](screenshots/04-runtime-env-override.png)

**Explanation:**
This screenshot demonstrates that environment variables defined in the image can be overridden when the container starts. The same `env-demo` image can therefore be used with different runtime configurations without rebuilding it.

---

## 2.2 Environment File

An external environment file was created:

```text
APP_NAME=ProductionApp
APP_VERSION=3.0
APP_ENV=staging
```

The variables were supplied using:

```bash
podman run --rm --env-file=app.env env-demo
```

### Screenshot 5 — Environment File Configuration

![Environment file override](screenshots/05-env-file-override.png)

**Explanation:**
This screenshot shows the `app.env` configuration and its use with `--env-file`. It demonstrates how multiple environment variables can be supplied externally instead of specifying each variable individually on the command line.

---

# Task 3: Inspect Environment Variables in a Running Container

## 3.1 Start the Container

Because the original application command exits after printing its message, a long-running command was used for inspection:

```bash
podman rm -f env-container 2>/dev/null || true

podman run -d \
  --name env-container \
  env-demo \
  sh -c 'env; sleep 300'
```

## 3.2 Inspect Environment Variables

The variables were inspected with:

```bash
podman exec env-container env | grep -E '^(APP_NAME|APP_VERSION|APP_ENV)='
```

The container configuration was also inspected with:

```bash
podman inspect env-container \
  --format '{{range .Config.Env}}{{println .}}{{end}}'
```

### Screenshot 6 — Environment Variable Inspection

![Environment variable inspection](screenshots/06-environment-inspection.png)

**Explanation:**
This screenshot provides evidence that the environment variables are actually present inside the running container. It demonstrates two useful troubleshooting methods: inspecting the live process environment with `podman exec` and checking the container configuration with `podman inspect`.

---

# Task 4: Build-Time Variables with ARG

## 4.1 Modify the Containerfile

The Containerfile was changed to demonstrate a build-time argument:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal

ARG APP_BUILD_NUMBER
ENV APP_BUILD=$APP_BUILD_NUMBER

CMD echo "Build number: $APP_BUILD"
```

`ARG` is available during image construction, while `ENV` makes the resulting value available when the container runs.

---

## 4.2 Build with ARG

The image was built with:

```bash
podman build --no-cache \
  --build-arg APP_BUILD_NUMBER=42 \
  -t arg-demo .
```

### Screenshot 7 — ARG Build

![ARG build](screenshots/07-arg-build.png)

**Explanation:**
This screenshot shows the `arg-demo` image being built with `APP_BUILD_NUMBER=42`. The `--build-arg` option supplies the value during the image build process.

---

## 4.3 Run the ARG Image

The resulting image was executed with:

```bash
podman run --rm arg-demo
```

The resulting output is:

```text
Build number: 42
```

### Screenshot 8 — ARG Runtime Verification

![ARG runtime verification](screenshots/08-arg-runtime.png)

**Explanation:**
This screenshot verifies that the build-time argument was successfully transferred into the `APP_BUILD` environment variable and is available when the container runs.

---

# ENV vs ARG

| Feature                  | `ENV`                     | `ARG`                        |
| ------------------------ | ------------------------- | ---------------------------- |
| Available during build   | Yes                       | Yes                          |
| Available at runtime     | Yes                       | No, unless assigned to `ENV` |
| Runtime override         | Yes                       | No                           |
| Build-time configuration | Possible                  | Primary purpose              |
| Typical use              | Application configuration | Build parameters             |

## Configuration Flow

```text
             Containerfile
                  │
          ┌───────┴────────┐
          │                │
         ENV              ARG
          │                │
          │          Build-time value
          │                │
          ↓                ↓
     Runtime config    ENV assignment
          │                │
          └───────┬────────┘
                  ↓
             Container
```

---

# Key Findings

### Finding 1 — ENV Provides Runtime Configuration

The `ENV` instruction successfully provided default configuration values to the running container.

### Finding 2 — Runtime Overrides Work

The `-e` option allowed individual variables to be changed without rebuilding the image.

### Finding 3 — Environment Files Simplify Configuration

The `--env-file` option allowed several configuration variables to be supplied from one file.

### Finding 4 — Environment Variables Can Be Inspected

`podman exec` and `podman inspect` provided ways to verify the container's environment.

### Finding 5 — ARG Provides Build-Time Configuration

`ARG` allowed a value to be supplied during the image build using `--build-arg`.

### Finding 6 — ARG and ENV Have Different Purposes

`ARG` is intended primarily for build-time configuration, while `ENV` provides runtime environment variables.

### Finding 7 — Build Arguments Can Become Runtime Variables

The lab demonstrated the pattern:

```text
ARG → ENV → Running Container
```

---

# Security Considerations

Environment variables should not automatically be treated as secure storage.

Avoid putting sensitive information such as:

* Passwords
* API keys
* Access tokens
* Private credentials
* Encryption keys

directly into Containerfiles or images.

For container orchestration platforms such as Kubernetes and OpenShift:

* Use ConfigMaps for non-sensitive configuration.
* Use Secrets for sensitive configuration.
* Keep environment-specific configuration outside immutable container images.

---

# Practical Applications

Environment variables are commonly used to configure:

```text
APP_ENV
APP_VERSION
LOG_LEVEL
DATABASE_HOST
DATABASE_PORT
SERVICE_URL
```

The same container image can then be deployed to different environments with different runtime configuration.

For example:

```text
Development → APP_ENV=development
Staging     → APP_ENV=staging
Production  → APP_ENV=production
```

The image itself does not need to be rebuilt for each environment.

---

# Conclusion

This lab demonstrated practical environment-variable management with Podman.

The exercises showed how:

* `ENV` defines default runtime configuration.
* `-e` overrides variables at container startup.
* `--env-file` loads multiple variables from an external file.
* `podman exec` can inspect variables inside a running container.
* `podman inspect` can inspect container configuration.
* `ARG` provides build-time configuration.
* `ARG` values can be transferred to `ENV` when runtime access is required.

These techniques are important when building reusable containe
