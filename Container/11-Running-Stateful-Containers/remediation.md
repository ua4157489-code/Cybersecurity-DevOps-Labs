# Lab 11: Remediation and Best Practices

## 1. Protect Database Credentials

The lab uses passwords directly in environment variables for demonstration purposes.

For production environments, avoid hard-coding credentials in scripts or public configuration files.

Use appropriate secret-management mechanisms such as:

* Podman secrets
* Kubernetes/OpenShift Secrets
* External secret-management systems

## 2. Protect Database Storage

Database files stored on the host should have restrictive permissions.

Limit access to the directories containing:

```text
mysql-data/
pg-data/
```

Only authorized users and processes should be able to access database files.

## 3. Back Up Persistent Data

Persistent storage protects data from container removal, but it is not a backup.

Implement regular backups for important database workloads.

Backups should be tested periodically to ensure that restoration actually works.

## 4. Avoid Exposing Databases Unnecessarily

The lab publishes:

```text
3306:3306
5432:5432
```

For production environments, database ports should not be exposed publicly unless there is a specific requirement.

Prefer private container networks and controlled access.

## 5. Use Least Privilege

Database users should have only the permissions required by the application.

Avoid using the database root or administrator account for normal application operations.

## 6. Protect Bind Mounts

Bind mounts expose host filesystem locations to containers.

Only mount directories that are required by the application.

Avoid mounting sensitive host directories into containers.

## 7. Use Read-Only Mounts Where Appropriate

When an application only needs read access, use a read-only mount where supported.

This reduces the ability of a compromised container process to modify host data.

## 8. Monitor Persistent Storage

Monitor:

* Disk usage
* Database growth
* File permissions
* Container health
* Database logs
* Backup status

Unexpected growth or access can indicate operational or security problems.

## 9. Plan Data Lifecycle

Container deletion and data deletion are separate operations.

Before removing persistent storage, verify:

* The data is no longer required.
* A backup exists if needed.
* No active container depends on the storage.
* The deletion is authorized.

## 10. Production Stateful Container Design

A production database deployment should consider:

```text
Database
   │
   ├── Persistent Storage
   ├── Access Control
   ├── Secrets Management
   ├── Backups
   ├── Monitoring
   ├── Network Security
   └── Recovery Strategy
```

## Conclusion

Stateful containers require more than simply running a database image.

Persistent storage, credentials, permissions, network exposure, backups, monitoring, and recovery procedures should all be considered when deploying databases in production environments.
