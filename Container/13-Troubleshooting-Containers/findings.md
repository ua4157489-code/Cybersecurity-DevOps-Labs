# Lab 13: Troubleshooting Containers — Findings

## 1. Container Runtime

The Nginx container was running successfully.

```text
Container: nginx-test
Status: running
Exit Code: 0
Restart Count: 0
OOMKilled: false
```

No runtime error was reported.

## 2. Nginx Service

The deployed image was:

```text
docker.io/library/nginx:alpine
```

The running Nginx version was:

```text
nginx/1.31.6
```

Nginx successfully initialized its worker processes.

## 3. Log Analysis

Container logs showed successful startup messages followed by HTTP access-log entries.

Observed requests returned:

```text
HTTP 200
```

This confirmed that Nginx was serving requests successfully.

## 4. Resource Usage

The observed resource usage was:

```text
CPU:       0.09%
Memory:    4.067 MB / 16.6 GB
Memory %:  0.02%
PIDs:      5
```

The container was not experiencing obvious resource exhaustion.

## 5. Networking

The container published:

```text
0.0.0.0:8080 -> 80/tcp
```

Podman was using:

```text
Network Mode: slirp4netns
```

The inspected container `IPAddress` field was empty. This was consistent with the rootless networking configuration observed during the lab.

## 6. Process Inspection

Inside the container, the following processes were observed:

* Nginx master process.
* Four Nginx worker processes.
* Interactive shell.
* `ps` troubleshooting command.

This confirmed that the Nginx service was actively running.

## 7. Configuration Validation

The Nginx configuration was tested with:

```bash
nginx -t
```

The result was:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

No configuration syntax errors were identified.

## 8. Internal Connectivity

The command:

```bash
wget -qO- http://localhost
```

successfully returned the Nginx welcome page.

This confirmed that the service was reachable from within the container.

## 9. External Connectivity

The host-side request:

```bash
curl -I http://localhost:8080
```

returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.31.6
Content-Type: text/html
Content-Length: 896
```

This confirmed successful host-to-container connectivity.

## 10. Overall Finding

The troubleshooting investigation found no active failure in the Nginx container.

The complete troubleshooting chain—from runtime state and logs through application configuration and network connectivity—showed a functioning containerized web service.
