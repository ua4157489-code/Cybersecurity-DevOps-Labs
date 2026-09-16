# Lab 12 Methodology

## 1. Database Deployment

A MySQL 8.0 container was deployed using Podman with environment variables defining the root password, database, and application user.

The container was allowed to complete its initial database initialization before further operations were performed.

## 2. Data Creation

A database named `testdb` was initialized with an `employees` table.

Two records were inserted:

```text
John Doe
Jane Smith
```

The data was queried to establish the original state before backup.

## 3. Backup Creation

The MySQL `mysqldump` utility was executed inside the running container.

The output was redirected to the host filesystem as:

```text
testdb_dump.sql
```

The dump was inspected to confirm that it contained both the `employees` table definition and the inserted records.

## 4. Persistent Backup Storage

A Podman named volume named `backup-vol` was created.

A temporary Alpine container was used to mount the volume, allowing the SQL dump to be copied into persistent storage.

The temporary container was removed after the copy operation.

The backup file was then verified directly from the named volume.

## 5. Container Replacement

The original MySQL container was removed after the backup had been verified.

A new MySQL container named `mysql-restore` was deployed with the same database configuration.

This simulated recovery of a database after loss of the original container.

## 6. Data Restoration

The SQL dump was retrieved from `backup-vol` and copied to the host working directory.

The dump was then passed into the new MySQL container using standard input:

```bash
podman exec -i mysql-restore \
  mysql -u root -predhat testdb < testdb_dump.sql
```

## 7. Recovery Validation

The restored database was queried after the import.

Validation confirmed:

* The `employees` table existed.
* `John Doe` was present.
* `Jane Smith` was present.
* The backup file remained available in `backup-vol`.

## 8. Evidence Collection

The practical workflow was completed before screenshots were collected.

Five screenshots were selected to document the key stages:

1. MySQL container and original data
2. Database dump
3. Backup volume
4. Restored data
5. Final verification

This keeps the documentation focused on meaningful evidence rather than individual commands.
