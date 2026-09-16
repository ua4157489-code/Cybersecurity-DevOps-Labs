# Lab 11: Commands

## 1. Lab Setup

```bash
cd ~/Alrazzaq_Labs/Container/11-Running-Stateful-Containers

mkdir -p screenshots

podman --version
pwd
```

## 2. Create MySQL Storage

```bash
mkdir -p mysql-data
ls -ld mysql-data
```

## 3. Start MySQL

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=user123 \
  -v "$(pwd)/mysql-data:/var/lib/mysql:Z" \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

Verify:

```bash
podman ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

## 4. Check MySQL Logs

```bash
podman logs mysql-db
```

Or:

```bash
podman logs --tail 30 mysql-db
```

## 5. Test MySQL Readiness

```bash
podman exec mysql-db mysqladmin ping -uroot -predhat123
```

## 6. Connect to MySQL

```bash
podman exec -it mysql-db mysql -u testuser -puser123 testdb
```

Inside MySQL:

```sql
SELECT DATABASE();

SHOW TABLES;

CREATE TABLE lab_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message VARCHAR(255)
);

INSERT INTO lab_data (message)
VALUES ('Persistent test data');

SELECT * FROM lab_data;

exit;
```

## 7. Remove MySQL Container

```bash
podman stop mysql-db
podman rm mysql-db
```

Verify:

```bash
podman ps -a
```

## 8. Recreate MySQL

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=user123 \
  -v "$(pwd)/mysql-data:/var/lib/mysql:Z" \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

Verify:

```bash
podman ps
```

## 9. Verify MySQL Persistence

```bash
podman exec mysql-db mysqladmin ping -uroot -predhat123
```

```bash
podman exec mysql-db \
  mysql -u testuser -puser123 testdb \
  -e "SELECT * FROM lab_data;"
```

---

# PostgreSQL

## 10. Remove MySQL Before PostgreSQL

```bash
podman rm -f mysql-db
```

## 11. Create PostgreSQL Storage

```bash
mkdir -p pg-data
ls -ld pg-data
```

## 12. Start PostgreSQL

```bash
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_DB=testdb \
  -v "$(pwd)/pg-data:/var/lib/postgresql/data:Z" \
  -p 5432:5432 \
  docker.io/library/postgres:13
```

Verify:

```bash
podman ps -a
```

## 13. Check PostgreSQL Logs

```bash
podman logs --tail 30 postgres-db
```

## 14. Create PostgreSQL Table

```bash
podman exec postgres-db \
  psql -U testuser -d testdb \
  -c "CREATE TABLE lab_data (id SERIAL PRIMARY KEY, message TEXT);"
```

Insert data:

```bash
podman exec postgres-db \
  psql -U testuser -d testdb \
  -c "INSERT INTO lab_data (message) VALUES ('Postgres persistent data');"
```

Verify:

```bash
podman exec postgres-db \
  psql -U testuser -d testdb \
  -c "SELECT * FROM lab_data;"
```

## 15. Remove PostgreSQL Container

```bash
podman stop postgres-db
podman rm postgres-db
```

## 16. Recreate PostgreSQL

```bash
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_DB=testdb \
  -v "$(pwd)/pg-data:/var/lib/postgresql/data:Z" \
  -p 5432:5432 \
  docker.io/library/postgres:13
```

Verify:

```bash
podman ps
```

## 17. Verify PostgreSQL Persistence

```bash
podman exec postgres-db \
  psql -U testuser -d testdb \
  -c "SELECT * FROM lab_data;"
```

## 18. Inspect Host Storage

```bash
du -sh mysql-data pg-data
```

```bash
find mysql-data -maxdepth 2 -type f | head
```

```bash
find pg-data -maxdepth 2 -type f | head
```

## 19. Cleanup

Only run after all screenshots and documentation are complete:

```bash
podman rm -f postgres-db
rm -rf mysql-data pg-data
```
