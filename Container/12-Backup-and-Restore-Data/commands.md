# Lab 12 Commands: Backup and Restore Data

## 1. Verify Podman

```bash
podman --version
```

## 2. Create MySQL Container

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  docker.io/library/mysql:8.0
```

Verify:

```bash
podman ps -f name=mysql-db
```

Check MySQL readiness:

```bash
podman exec mysql-db mysqladmin ping -uroot -predhat
```

## 3. Create Database Data

```bash
podman exec mysql-db mysql -u root -predhat testdb -e "
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO employees (name)
VALUES ('John Doe'), ('Jane Smith');

SELECT * FROM employees;
"
```

## 4. Create Database Dump

```bash
podman exec mysql-db \
  mysqldump -u root -predhat testdb > testdb_dump.sql
```

Verify:

```bash
ls -lh testdb_dump.sql
grep -E "CREATE TABLE|INSERT INTO" testdb_dump.sql
```

## 5. Create Backup Volume

```bash
podman volume create backup-vol
```

Verify:

```bash
podman volume ls
```

## 6. Copy Dump to Backup Volume

Create a temporary container:

```bash
podman create \
  --name backup-helper \
  -v backup-vol:/backup \
  alpine
```

Copy the dump:

```bash
podman cp testdb_dump.sql backup-helper:/backup/
```

Remove the temporary container:

```bash
podman rm backup-helper
```

Verify:

```bash
podman run --rm \
  -v backup-vol:/backup \
  alpine \
  ls -lh /backup/
```

## 7. Remove Original Database Container

```bash
podman rm -f mysql-db
```

## 8. Create Restore Container

```bash
podman run -d \
  --name mysql-restore \
  -e MYSQL_ROOT_PASSWORD=redhat \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  docker.io/library/mysql:8.0
```

Check readiness:

```bash
podman exec mysql-restore mysqladmin ping -uroot -predhat
```

## 9. Retrieve Backup

```bash
podman run --rm \
  -v backup-vol:/backup \
  -v "$(pwd):/restore:Z" \
  alpine \
  cp /backup/testdb_dump.sql /restore/testdb_dump.sql
```

Verify:

```bash
ls -lh testdb_dump.sql
```

## 10. Restore Database

```bash
podman exec -i mysql-restore \
  mysql -u root -predhat testdb < testdb_dump.sql
```

## 11. Verify Restored Data

```bash
podman exec mysql-restore \
  mysql -u root -predhat testdb \
  -e "SELECT * FROM employees;"
```

Verify tables:

```bash
podman exec mysql-restore \
  mysql -u root -predhat testdb \
  -e "SHOW TABLES;"
```

## 12. Final Backup Verification

```bash
podman run --rm \
  -v backup-vol:/backup \
  alpine \
  ls -lh /backup/
```

Check container state:

```bash
podman ps
```

Check volume:

```bash
podman volume inspect backup-vol
```
