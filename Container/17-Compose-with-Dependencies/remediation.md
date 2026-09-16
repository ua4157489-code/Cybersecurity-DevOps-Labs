# Lab 17 — Remediation and Security Recommendations

## 1. Purpose

This document provides operational and security recommendations for improving the containerized Flask and Redis environment demonstrated in Lab 17.

The lab was successfully completed, but several improvements would be required before using a similar architecture in a production environment.

---

# 2. Use a Production WSGI Server

## Observation

The Flask application currently runs using Flask's development server:

```text
WARNING: This is a development server.
Do not use it in a production deployment.
```

## Recommendation

Use a production WSGI server such as Gunicorn.

Example:

```bash
pip install gunicorn
```

Container command:

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

## Benefit

A production WSGI server provides a more appropriate process model for production workloads.

---

# 3. Protect Redis

## Observation

Redis was used as the shared backend.

## Recommendation

Redis should not be exposed directly to untrusted networks.

Production deployments should:

* Restrict Redis network access.
* Use authentication where appropriate.
* Apply firewall/network policies.
* Avoid unnecessary host-port publishing.
* Keep Redis on an internal application network.

## Benefit

This reduces the attack surface of the backend database.

---

# 4. Use Secrets for Sensitive Configuration

## Observation

The lab did not require sensitive credentials.

For production environments, database passwords or other secrets should not be stored directly in Compose files.

## Recommendation

Use:

* Container secrets
* External secret managers
* Environment injection from protected systems

Avoid committing credentials to Git repositories.

---

# 5. Pin Image Versions

## Observation

The lab uses floating image tags such as:

```text
redis:alpine
```

and:

```text
python:3.12-alpine
```

## Recommendation

Production deployments should use controlled image versions and preferably immutable image references.

For example:

```yaml
image: redis:<tested-version>
```

## Benefit

Pinning versions improves:

* Reproducibility
* Change control
* Deployment consistency
* Supply-chain management

---

# 6. Scan Container Images

Before deployment, scan application and base images for vulnerabilities.

Recommended workflow:

```text
Build image
    ↓
Scan image
    ↓
Review vulnerabilities
    ↓
Update dependencies
    ↓
Rebuild
    ↓
Retest
    ↓
Deploy
```

Container images should be regularly updated to address known vulnerabilities.

---

# 7. Implement Health Checks

The Flask application already provides:

```text
/health
```

which verifies Redis connectivity.

This endpoint can be integrated into a container health-check mechanism.

Example concept:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
```

The exact health-check implementation should match the tools available inside the final application image.

---

# 8. Apply Resource Limits

Multiple application replicas can consume increasing amounts of CPU and memory.

Production deployments should define appropriate:

* CPU limits
* Memory limits
* Process limits

This prevents a faulty or overloaded container from consuming excessive host resources.

---

# 9. Restrict Network Exposure

The development configuration publishes:

```text
5000:5000
```

For production environments, application exposure should normally be controlled through an appropriate reverse proxy or ingress layer.

Redis should remain accessible only to services that require it.

---

# 10. Centralize Logging

Application and Redis logs should be collected centrally.

Recommended architecture:

```text
Containers
    ↓
Container Runtime Logs
    ↓
Log Collector
    ↓
Central Logging Platform
    ↓
Monitoring / SIEM
```

This allows administrators and security teams to investigate application errors and suspicious activity.

---

# 11. Monitor Replicas

When multiple application replicas are deployed, monitor:

* Container availability
* CPU utilization
* Memory utilization
* Request rates
* Response times
* Error rates
* Redis availability
* Redis connection failures

Monitoring helps identify resource exhaustion and application failures.

---

# 12. Use Least Privilege

Containers should run with only the privileges required by the application.

Avoid unnecessary:

```text
--privileged
```

settings.

Also avoid unnecessary host filesystem mounts and excessive Linux capabilities.

---

# 13. Protect the Container Supply Chain

For production use:

* Use trusted base images.
* Scan dependencies.
* Review image provenance.
* Keep build definitions under version control.
* Use controlled registries.
* Avoid untrusted images.
* Regularly rebuild outdated images.

---

# 14. Secure Configuration Management

Configuration should be separated from application code where appropriate.

For example:

```text
Application Code
        +
Configuration
        +
Secrets
        +
Environment-specific Settings
```

This makes deployments easier to manage across development, testing, and production environments.

---

# 15. Backup Redis Data Where Required

If Redis contains important persistent application state, an appropriate backup strategy should be implemented.

The backup strategy should define:

* Backup frequency
* Retention period
* Storage location
* Restore procedure
* Recovery testing

A backup that has never been restored should not be assumed to be reliable.

---

# 16. Recommended Production Architecture

A more production-oriented architecture could be:

```text
                  Internet
                      |
                Reverse Proxy
                      |
              +-------+-------+
              |               |
          Flask App       Flask App
          Replica 1       Replica 2
              |               |
              +-------+-------+
                      |
                   Redis
                      |
              Protected Network
```

Additional monitoring and centralized logging can be added around the application and infrastructure.

---

# 17. Final Recommendations

| Area         | Recommendation                                 |
| ------------ | ---------------------------------------------- |
| Flask        | Use Gunicorn or another production WSGI server |
| Redis        | Keep private and restrict access               |
| Secrets      | Use secure secret management                   |
| Images       | Pin tested versions                            |
| Security     | Scan images and dependencies                   |
| Networking   | Minimize exposed ports                         |
| Health       | Maintain application health checks             |
| Resources    | Apply CPU/memory limits                        |
| Logging      | Centralize container logs                      |
| Monitoring   | Monitor application and Redis                  |
| Privileges   | Apply least privilege                          |
| Backups      | Implement and test backups                     |
| Supply Chain | Use trusted and verified images                |

These controls would strengthen the security, reliability, and maintainability of a production deployment based on the architecture demonstrated in this lab.
