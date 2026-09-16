# Lab 8: Findings

## Finding 1 — ENV Provides Runtime Configuration

The `ENV` instruction successfully defined:

```text
APP_NAME=MyApp
APP_VERSION=1.0
APP_ENV=development
```

These values were available when the image was executed.

Evidence:

```text
Running MyApp v1.0 in development mode
```

---

## Finding 2 — Runtime Overrides Work

Podman allowed environment variables to be overridden with the `-e` option.

For example:

```bash
-e APP_ENV="production"
-e APP_VERSION="2.0"
```

This allows the same image to be used with different runtime configurations.

---

## Finding 3 — Environment Files Simplify Configuration

The `app.env` file provided multiple configuration values:

```text
APP_NAME=ProductionApp
APP_VERSION=3.0
APP_ENV=staging
```

Using:

```bash
podman run --env-file=app.env env-demo
```

allows configuration to be maintained separately from the image.

---

## Finding 4 — Environment Variables Can Be Inspected

The running container's environment was inspected with:

```bash
podman exec env-container env
```

The same configuration could also be examined through:

```bash
podman inspect env-container
```

This is useful when troubleshooting container configuration.

---

## Finding 5 — ARG Is Build-Time Configuration

The `ARG` instruction was used for:

```text
APP_BUILD_NUMBER
```

The build command supplied:

```text
APP_BUILD_NUMBER=42
```

The value was then assigned to an environment variable.

---

## Finding 6 — ARG and ENV Have Different Lifecycles

`ARG` is primarily intended for image-build configuration, while `ENV` provides values available to the running container.

The lab demonstrated how they can be combined when a build-time value needs to become runtime configuration.

---

## Finding 7 — Configuration and Secrets Should Be Separated

Environment variables are useful for configuration but should not be considered a secure replacement for dedicated secret-management mechanisms.

Sensitive credentials should be handled through appropriate secret-management solutions.
