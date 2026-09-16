# Lab 13: Troubleshooting Containers — Commands

## 1. Create the Nginx Container

```bash
podman pull docker.io/library/nginx:alpine

podman run -d \
  --name nginx-test \
  -p 8080:80 \
  docker.io/library/nginx:alpine
```

## 2. Verify Container Status

```bash
podman ps
```

## 3. Test HTTP Service

```bash
curl http://localhost:8080
```

## 4. View Container Logs

```bash
podman logs nginx-test
```

## 5. Follow Logs in Real Time

```bash
podman logs -f nginx-test
```

Press `Ctrl+C` to stop following the logs.

## 6. Filter Logs by Time

```bash
podman logs --since 5m nginx-test
```

## 7. Display Last 10 Log Entries

```bash
podman logs --tail 10 nginx-test
```

## 8. Inspect Container

```bash
podman inspect nginx-test
```

## 9. Inspect Runtime State

```bash
podman inspect nginx-test \
  --format 'Status={{.State.Status}} ExitCode={{.State.ExitCode}} Error={{.State.Error}} OOMKilled={{.State.OOMKilled}} RestartCount={{.RestartCount}}'
```

## 10. Check Port Mapping

```bash
podman port nginx-test
```

## 11. Check Network Mode

```bash
podman inspect nginx-test \
  --format 'NetworkMode={{.HostConfig.NetworkMode}}'
```

## 12. Check Resource Usage

```bash
podman stats --no-stream nginx-test
```

## 13. Open Container Shell

```bash
podman exec -it nginx-test /bin/sh
```

## 14. Check Processes

```bash
ps aux
```

## 15. Inspect Nginx Configuration

```bash
cat /etc/nginx/nginx.conf
```

## 16. Test Internal Connectivity

```bash
wget -qO- http://localhost
```

## 17. Validate Nginx Configuration

```bash
nginx -t
```

## 18. Exit Container

```bash
exit
```

## 19. Verify Host Connectivity

```bash
curl -I http://localhost:8080
```

## 20. Cleanup

```bash
podman stop nginx-test
podman rm nginx-test
```
