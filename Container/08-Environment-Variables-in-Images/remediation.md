# Lab 8: Remediation and Best Practices

## 1. Keep Configuration Outside the Image When Appropriate

Application configuration that changes between environments should generally be supplied at runtime rather than requiring a new image for every environment.

Examples include:

```text
APP_ENV
APP_VERSION
SERVICE_URL
LOG_LEVEL
```

## 2. Use Environment Files for Non-Sensitive Configuration

Environment files can simplify local development and testing:

```bash
podman run --env-file=app.env env-demo
```

Keep environment files properly protected and avoid committing sensitive values to Git.

## 3. Do Not Store Secrets Directly in Containerfiles

Avoid patterns such as:

```dockerfile
ENV DATABASE_PASSWORD="secret"
```

Secrets embedded in image configuration can become accessible to users who can inspect the image.

Use appropriate secret-management mechanisms instead.

## 4. Use ARG Carefully

`ARG` is useful for build-time values such as:

* Build numbers
* Version identifiers
* Feature flags used during image construction
* Build configuration

Do not use build arguments as a secure mechanism for protecting secrets.

## 5. Validate Runtime Configuration

When troubleshooting a container, inspect the actual environment:

```bash
podman exec <container> env
```

and:

```bash
podman inspect <container>
```

This helps identify incorrect or missing configuration.

## 6. Use Consistent Variable Naming

A consistent naming convention improves maintainability.

For example:

```text
APP_NAME
APP_VERSION
APP_ENV
APP_BUILD
```

## 7. Kubernetes and OpenShift Integration

In Kubernetes or OpenShift deployments:

* Use ConfigMaps for non-sensitive configuration.
* Use Secrets for sensitive configuration.
* Avoid hardcoding environment-specific values into container images.
* Keep container images reusable across development, staging, and production.

## Recommended Configuration Flow

```text
Container Image
      │
      ├── Default ENV
      │
      ↓
Runtime Configuration
      │
      ├── -e
      ├── --env-file
      ├── ConfigMap
      └── Secret
      │
      ↓
Application
```

## Final Recommendation

Use immutable container images whenever possible and inject environment-specific configuration at deployment or runtime. Keep secrets outside the image and use dedicated secret-management mechanisms for sensitive information.
