# Lab 12 Findings

## Finding 1 — MySQL Container Successfully Initialized

The MySQL 8.0.46 container initialized successfully and reported:

```text
mysqld is alive
```

The database service was ready to accept connections on port 3306.

## Finding 2 — Database Data Was Created

The `testdb` database contained the `employees` table.

The original records were:

```text
1    John Doe
2    Jane Smith
```

## Finding 3 — Database Dump Was Created

The database was successfully exported using `mysqldump`.

The resulting file:

```text
testdb_dump.sql
```

was approximately 2.0 KB.

The dump contained the `employees` table definition and both records.

## Finding 4 — Backup Was Stored Persistently

The Podman named volume:

```text
backup-vol
```

successfully stored:

```text
testdb_dump.sql
```

The file remained available after the original MySQL container was removed.

## Finding 5 — Database Was Successfully Restored

A new MySQL container named:

```text
mysql-restore
```

was created and the SQL dump was imported successfully.

The restored database contained:

```text
employees
```

with:

```text
1    John Doe
2    Jane Smith
```

## Finding 6 — Backup Integrity Was Verified

The backup volume was inspected after restoration and still contained the SQL dump.

This confirmed that the backup storage was independent of the original database container lifecycle.

## Overall Result

The complete backup and restore workflow was successful.

```text
Database
   ↓
mysqldump
   ↓
SQL Backup
   ↓
Podman Named Volume
   ↓
New MySQL Container
   ↓
Database Restore
   ↓
Data Verification```
