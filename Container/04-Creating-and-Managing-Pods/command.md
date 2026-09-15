# 💻 Command Reference — Lab 4

## 1. Environment Verification

```bash
podman --version
podman info --format 'Rootless={{.Host.Security.Rootless}} | Runtime={{.Host.OCIRuntime.Name}} | Network={{.Host.NetworkBackend}}'
podman images
```

## 2. Pull Images

```bash
podman pull docker.io/library/nginx:alpine
podman pull docker.io/library/redis:alpine
```

## 3. Create Pod

```bash
podman pod create --name demo-pod -p 8080:80
podman pod list
```

## 4. Start Nginx

```bash
podman run -d --pod demo-pod --name nginx-container docker.io/library/nginx:alpine
podman ps --pod
```

## 5. Test Nginx

```bash
curl http://localhost:8080
```

## 6. Start Redis

```bash
podman run -d --pod demo-pod --name redis-container docker.io/library/redis:alpine
podman ps --pod
```

## 7. Test Redis

```bash
podman exec redis-container redis-cli ping
```

Expected result:

```
PONG
```

## 8. Verify Shared Networking

```bash
podman exec nginx-container sh -c 'wget -qO- http://127.0.0.1:80 | head'
```

## 9. Install jq

```bash
sudo apt install -y jq
```

## 10. Inspect Pod

```bash
podman pod inspect demo-pod
```

A useful formatted inspection:

```bash
podman pod inspect demo-pod | jq '{
  id: .Id,
  name: .Name,
  status: .State,
  containers: .Containers
}'
```

## 11. Create Volume

```bash
podman volume create shared-vol
podman volume inspect shared-vol
```

## 12. Start Volume-Mounted Containers

```bash
podman run -d --pod demo-pod --name nginx2 -v shared-vol:/data docker.io/library/nginx:alpine
podman run -d --pod demo-pod --name redis2 -v shared-vol:/data docker.io/library/redis:alpine
```

## 13. Check Container State

```bash
podman ps -a --pod
```

## 14. Attempt Shared File Test

```bash
podman exec nginx2 touch /data/testfile
podman exec redis2 ls -la /data
```

Actual result:

```
Error: can only create exec sessions on running containers: container state improper
```

## 15. Troubleshoot Failed Containers

```bash
podman logs nginx2
podman logs redis2
podman inspect nginx2 --format '{{.State.Status}}'
podman inspect redis2 --format '{{.State.Status}}'
```

## 16. Cleanup

```bash
podman pod rm -f demo-pod
podman volume rm shared-vol
```

## 17. Final Verification

```bash
podman pod ps
podman ps -a
podman volume ls
```

All lab resources were successfully removed.

## ⚠️ Troubleshooting Notes

The following commands from the original task did not match the Podman 4.9.3 output/command behavior observed during the lab:

**`podman pod inspect demo-pod | jq '.Containers[].Names'`**
This returned null values because the inspected JSON structure did not contain the expected field at that location.

**`podman pod inspect demo-pod | jq '.[0].InfraConfig.NetworkOptions'`**
This failed because the inspection result was an object rather than an array.

**`podman port demo-pod`**
This failed because `podman port` expects a container name or ID rather than the pod name.

**`podman inspect nginx-container --format '{{.NetworkSettings.IPAddress}}'`**
This returned a blank value in the tested rootless Podman environment.
