# 💻 Command Reference — Lab 16

## 1. Enter the Lab Directory

```bash
cd ~/Alrazzaq_Labs/Container/16-Compose-Basics
```

## 2. Check Versions

```bash
podman --version
podman-compose --version
```

Verified:

```
Podman 4.9.3
podman-compose version 1.0.6
```

## 3. Verify Project Files

```bash
ls -la
cat podman-compose.yml
echo
cat html/index.html
```

## 4. Start the Compose Application

```bash
podman-compose up -d
```

## 5. List Running Containers

```bash
podman ps
```

Expected lab services:

```
16-compose-basics_web_1
16-compose-basics_db_1
```

## 6. Verify Web Port

```bash
podman port 16-compose-basics_web_1
```

Verified:

```
80/tcp -> 0.0.0.0:8080
```

## 7. Test Nginx

```bash
curl -i http://localhost:8080
```

Verified HTTP response:

```
HTTP/1.1 200 OK
Server: nginx/1.31.6
```

## 8. Verify PostgreSQL Readiness

```bash
podman exec 16-compose-basics_db_1 pg_isready -U postgres
```

Verified:

```
/var/run/postgresql:5432 - accepting connections
```

## 9. Verify PostgreSQL Version

```bash
podman exec 16-compose-basics_db_1 \
  psql -U postgres -c "SELECT version();"
```

Verified PostgreSQL:

```
13.23
```

## 10. Inspect Compose Network

```bash
podman network inspect 16-compose-basics_default
```

Verified:

```
Driver: bridge
Subnet: 10.89.1.0/24
Gateway: 10.89.1.1
DNS enabled: yes
```

## 11. Verify Bind Mount

```bash
podman exec 16-compose-basics_web_1 \
  cat /usr/share/nginx/html/index.html
```

## 12. Test Live Update

```bash
cat > html/index.html <<'EOF'
<h1>Hello from Podman Compose!</h1>
<p>Live bind mount update verified.</p>
EOF

curl http://localhost:8080
```

## 13. Stop and Remove Compose Containers

```bash
podman-compose down
```

## 14. Verify Cleanup

```bash
podman ps -a
```

The final container list was empty.

## ⚠️ Troubleshooting Notes

**`podman-compose down` did not remove the Compose network**

In the tested environment, the following networks remained after teardown:

```
16-compose-basics_default
podman
ubuntu_default
```

To manually remove the leftover network after confirming it is no longer needed:

```bash
podman network inspect 16-compose-basics_default
podman network rm 16-compose-basics_default
```
