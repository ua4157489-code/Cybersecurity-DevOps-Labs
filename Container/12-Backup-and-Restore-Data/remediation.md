# Lab 12 Remediation and Best Practices

Although the backup and restore workflow was successful, production environments require additional controls around database backups.

## 1. Protect Database Credentials

The lab used passwords directly in command-line arguments:

```bash
-predhat
```

This is acceptable for a controlled training environment but should not be used for sensitive production credentials.

### Recommendation

Use a secure secret-management mechanism such as:

* Podman secrets
* Environment-specific secret management
* Enterprise password/secret vaults

## 2. Encrypt Backups

SQL dumps may contain sensitive database information.

### Recommendation

Encrypt database backups both:

* At rest
* During transfer

Access to backup storage should also be restricted using least-privilege permissions.

## 3. Store Backups Off-Host

A local Podman volume protects against some container-level failures but does not protect against host-level loss.

### Recommendation

Maintain additional copies in separate storage locations such as:

* Dedicated backup servers
* Object storage
* Secure remote storage

## 4. Implement Backup Retention

A single backup is insufficient for production recovery requirements.

### Recommendation

Define retention periods and maintain multiple backup generations.

A practical strategy may include:

* Daily backups
* Weekly backups
* Monthly archival backups

The exact retention period should depend on organizational requirements.

## 5. Test Restores Regularly

Creating a backup does not guarantee that it can be restored successfully.

### Recommendation

Perform scheduled restore tests and verify:

* Database availability
* Table structure
* Record integrity
* Application functionality

## 6. Monitor Backup Jobs

Backup failures should generate an alert rather than remain unnoticed.

### Recommendation

Monitor:

* Backup job status
* Backup file size
* Backup age
* Storage capacity
* Restore-test results

## 7. Apply Least Privilege

The database account used for applications should not normally have unnecessary administrative privileges.

### Recommendation

Use separate accounts for:

* Application access
* Database administration
* Backup operations

## 8. Secure Backup Volume Access

The `backup-vol` volume contains database information and should be treated as sensitive storage.

### Recommendation

Restrict access to the volume and ensure that only authorized users and services can access backup files.

## 9. Maintain Recovery Documentation

Recovery procedures should be documented and tested.

The procedure demonstrated in this lab provides a basic recovery model:

```text
Create Dump
    ↓
Store Backup
    ↓
Create Replacement Database
    ↓
Restore Dump
    ↓
Verify Data
```

## Conclusion

The lab demonstrated successful database backup and recovery. For production use, the workflow should be extended with encryption, secure credential handling, off-host storage, retention policies, monitoring, access control, and regular restore testing.
