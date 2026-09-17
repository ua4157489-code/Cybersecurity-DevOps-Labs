# Lab 19 — Remediation and Security Hardening

## Overview

The lab successfully demonstrated a functional MySQL StatefulSet. However, the configuration was intentionally simplified for laboratory purposes.

A production deployment should apply additional Kubernetes, database, storage, and network security controls.

## 1. Store Credentials in Kubernetes Secrets

### Current Lab Configuration

The MySQL root password was directly defined in the StatefulSet:

```yaml
env:
- name: MYSQL_ROOT_PASSWORD
  value: "password"
```

This exposes the credential in the manifest and Kubernetes object configuration.

### Recommended Remediation

Use a Kubernetes Secret instead:

```bash
kubectl create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD='<strong-password>'
```

Reference the Secret from the StatefulSet:

```yaml
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: MYSQL_ROOT_PASSWORD
```

Production credentials should never be committed to Git repositories.

## 2. Avoid Using the Root Database Account for Applications

The lab uses the MySQL root account for verification.

Production applications should use a dedicated database account with only the permissions required by the application.

Recommended approach:

```text
Application
    ↓
Dedicated MySQL User
    ↓
Required Database Permissions
```

Avoid granting applications unnecessary administrative privileges.

## 3. Pin Container Images

The lab uses:

```text
mysql:8.0
```

A production deployment should use a specific tested version or image digest.

For example:

```yaml
image: mysql:<tested-version>
```

or an immutable image digest.

This prevents unexpected image changes from affecting deployments.

## 4. Configure Resource Requests and Limits

The lab does not specify CPU or memory resources.

Production workloads should define resource requests and appropriate limits.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

Values should be based on actual workload requirements.

## 5. Add Health Probes

Production MySQL Pods should use appropriate readiness and liveness checks.

A readiness probe prevents traffic from being directed to a database instance before it is ready.

A liveness probe can help Kubernetes detect an unhealthy container.

Probe configuration should be tested carefully because an overly aggressive liveness probe can cause unnecessary restarts.

## 6. Use Production-Appropriate Storage

The lab uses Kind's local-path StorageClass:

```text
standard
rancher.io/local-path
```

This is appropriate for the local lab environment.

Production deployments should use a reliable storage backend that provides the required:

* Availability
* Durability
* Performance
* Backup capability
* Encryption
* Recovery characteristics

## 7. Implement Backups

PersistentVolume storage alone should not be treated as a complete database backup strategy.

Production MySQL deployments should implement:

* Scheduled database backups
* Backup retention
* Off-cluster backup storage
* Restore testing
* Recovery procedures

A backup should periodically be restored in a separate environment to verify that it is actually usable.

## 8. Protect Network Access

The MySQL database should not be unnecessarily exposed outside the Kubernetes cluster.

A NetworkPolicy can restrict which Pods are allowed to connect to port `3306`.

Example concept:

```text
Application Pods
       │
       ▼
    MySQL 3306
       ▲
       │
Other Pods ──X──> MySQL
```

Only authorized application workloads should be permitted to reach the database.

## 9. Encrypt Database Traffic

Production database communication should use TLS where required.

This protects credentials and database traffic from interception within the network.

TLS configuration should be combined with appropriate certificate management and rotation.

## 10. Protect Persistent Data

Production storage should use appropriate security controls such as:

* Encryption at rest
* Access controls
* Storage permissions
* Backup encryption
* Restricted administrative access

The exact implementation depends on the Kubernetes platform and storage provider.

## 11. Use a Non-Root Container Security Model

The lab uses the standard MySQL container image and does not define an explicit Pod security context.

Production deployments should follow the security requirements of the selected image and platform, including:

* Least privilege
* Restricted capabilities
* Appropriate filesystem permissions
* Non-root execution where supported
* Read-only filesystem where compatible with the application

Database images must be tested before applying restrictive security settings because MySQL requires writable directories for normal operation.

## 12. Monitor PVC Capacity

Persistent databases can consume storage continuously.

Production monitoring should track:

```text
PVC capacity
Disk utilization
Database growth
IO performance
Backup status
```

Alerts should be configured before the database approaches its storage limit.

## 13. Implement Monitoring and Logging

Production StatefulSets should be monitored for:

* Pod restarts
* MySQL availability
* Replication or clustering health where applicable
* Storage utilization
* Query performance
* Error logs
* Authentication failures
* Resource consumption

Logs should be integrated with the organization's centralized logging or SIEM platform where appropriate.

## 14. Restrict Kubernetes Access

Access to the StatefulSet, Secrets, PVCs, and database Pods should follow Kubernetes RBAC principles.

Users and service accounts should receive only the permissions required for their role.

Avoid granting unnecessary:

```text
cluster-admin
```

privileges.

## 15. Protect Configuration in Git

The following should not be committed to a public repository:

* Real database passwords
* Production credentials
* Private keys
* TLS private keys
* Cloud credentials
* Sensitive connection strings

For this lab repository, use example values only and keep production secrets outside Git.

## 16. Define Update and Recovery Strategy

Stateful applications require careful planning for:

* Application updates
* Database upgrades
* Rollbacks
* Storage recovery
* Node failures
* Pod failures

Before upgrading MySQL, test the upgrade procedure and backup restoration process in a non-production environment.

## 17. Lab Configuration vs Production Configuration

| Area          | Lab                | Production Recommendation                      |
| ------------- | ------------------ | ---------------------------------------------- |
| Password      | Plaintext example  | Kubernetes Secret / external secret management |
| Image         | `mysql:8.0`        | Pinned version/digest                          |
| Storage       | Kind local-path    | Production storage backend                     |
| Backups       | Not configured     | Automated tested backups                       |
| NetworkPolicy | Not configured     | Restrict database access                       |
| TLS           | Not configured     | Encrypt database traffic where required        |
| Resources     | Not configured     | Requests and limits                            |
| Probes        | Not configured     | Readiness/liveness checks                      |
| Monitoring    | Basic kubectl      | Centralized monitoring                         |
| RBAC          | Default lab access | Least-privilege RBAC                           |
| Database user | Root               | Dedicated application user                     |

## Conclusion

The Lab 19 configuration was intentionally designed for learning StatefulSet behavior rather than production deployment.

The StatefulSet, persistent storage, headless Service, stable identity, DNS, and Pod recreation behavior were successfully verified.

Before using a similar architecture for a production database, the deployment should be hardened with secure secret management, least-privilege access, controlled networking, reliable storage, backups, monitoring, resource controls, health checks, and a tested recovery strategy.
