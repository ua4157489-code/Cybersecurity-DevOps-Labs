# Lab 11: Methodology

## 1. Environment Preparation

A dedicated directory was created for the stateful container lab.

Podman was verified before starting the database workloads.

## 2. MySQL Storage Configuration

A host directory named `mysql-data` was created.

This directory was used as persistent storage for the MySQL database.

The directory was mounted into the MySQL container at:

```text
/var/lib/mysql
```

The `:Z` SELinux labeling option was used with the Podman bind mount.

## 3. MySQL Initialization

The MySQL container was configured with:

* Root password
* Database name
* Database username
* Database password

The container was exposed through port `3306`.

Container status and logs were checked to verify startup.

## 4. MySQL Data Creation

A table named `lab_data` was created.

A test record containing:

```text
Persistent test data
```

was inserted.

The table was queried to verify that the record existed.

## 5. MySQL Lifecycle Test

The MySQL container was stopped and removed.

The host-side `mysql-data` directory was preserved.

A new MySQL container was then created using the same persistent directory.

The existing table and record were queried again.

The data remained available, demonstrating persistence across container replacement.

---

## 6. PostgreSQL Storage Configuration

A separate host directory named `pg-data` was created.

This directory was mounted into PostgreSQL at:

```text
/var/lib/postgresql/data
```

## 7. PostgreSQL Initialization

A PostgreSQL 13 container was started with:

* Database name: `testdb`
* Database user: `testuser`
* Persistent host storage
* Port `5432`

Container status and logs were checked.

## 8. PostgreSQL Data Creation

A `lab_data` table was created.

A test record containing:

```text
Postgres persistent data
```

was inserted.

The table was queried to confirm successful storage.

## 9. PostgreSQL Lifecycle Test

The PostgreSQL container was stopped and removed.

The `pg-data` directory remained on the host.

A replacement PostgreSQL container was then created using the same storage directory.

The previously created data was queried again and remained available.

## 10. Host Storage Validation

The MySQL and PostgreSQL storage directories were inspected from the host.

This provided additional evidence that database state was stored outside the individual container filesystem.

## Result

The methodology demonstrated the complete lifecycle of a stateful container:

```text
Create Storage
      ↓
Start Database Container
      ↓
Create Database Data
      ↓
Remove Container
      ↓
Recreate Container
      ↓
Reuse Persistent Storage
      ↓
Verify Original Data
```
