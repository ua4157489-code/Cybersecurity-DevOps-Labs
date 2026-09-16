# 🛡️ Remediation & Best Practices — Lab 16

## 1. Protect Database Credentials

**Issue**

The lab uses a plaintext password in the Compose file:

```yaml
POSTGRES_PASSWORD: example
```

**Recommendation**

For production, use secrets or a secure credential-management mechanism instead of committing passwords to source control. Options include:

- Podman/Docker secrets
- Environment files excluded from version control (`.env` + `.gitignore`)
- An external secrets manager (e.g. Vault, AWS Secrets Manager)

## 2. Avoid Unnecessary Port Exposure

Only the required application ports should be published. PostgreSQL does not need a host port in this lab, which limits direct host exposure. Keep it that way unless external database access is explicitly required.

## 3. Pin Image Versions

Instead of relying on floating tags where possible, use controlled image versions or digests:

```yaml
image: docker.io/library/postgres:13.23
```

Update pinned versions through a deliberate maintenance process rather than automatically picking up `latest`.

## 4. Keep Images Updated

Regularly pull and review updated base images to receive security patches:

```bash
podman pull docker.io/library/nginx:alpine
podman pull docker.io/library/postgres:13
```

## 5. Remove Unused Networks

The tested `podman-compose down` removed containers but left the project network. If the network is no longer required, verify it is unused and remove it manually:

```bash
podman network inspect 16-compose-basics_default
podman network rm 16-compose-basics_default
```

Only remove the network after confirming that no required containers depend on it.

## 6. Use Production-Appropriate Secrets

For real deployments, avoid storing sensitive credentials in:

- Git repositories
- Public Compose files
- Shell history
- Container image layers

Use an appropriate secret-management solution instead.

## 7. Apply Least Privilege

Run services with the minimum privileges, permissions, and network exposure required for their function. Avoid `--privileged` containers unless explicitly required.

## 8. Monitor Container Health

Production Compose deployments should include appropriate health checks and monitoring so service failures can be detected quickly, for example:

```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "postgres"]
  interval: 30s
  timeout: 5s
  retries: 3
```

## 9. Maintain Evidence-Based Documentation

For future labs:

- Run the command.
- Verify the output.
- Capture the screenshot.
- Record actual behavior (including unexpected results, such as networks not being removed).
- Apply remediation.
- Retest.
- Document only verified results.

## 10. Final Recommendation

The lab configuration is suitable for demonstrating Compose fundamentals. Production deployments should additionally address secret management, image pinning, network exposure, health checks, and lifecycle/cleanup management.
