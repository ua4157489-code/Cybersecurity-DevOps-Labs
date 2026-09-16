# Lab 13: Troubleshooting Containers — Methodology

## 1. Establish the Baseline

A standard Nginx Alpine container was deployed with Podman and exposed on host port `8080`.

```bash
podman run -d \
  --name nginx-test \
  -p 8080:80 \
  docker.io/library/nginx:alpine
```

The initial container state was verified with `podman ps`, followed by an HTTP request to confirm service availability.

## 2. Analyze Logs

Container logs were reviewed using `podman logs`.

The troubleshooting process included:

* Full container logs.
* Logs from the previous five minutes.
* The last ten log entries.
* Real-time log monitoring.

The logs confirmed successful Nginx initialization and recorded HTTP requests with status code `200`.

## 3. Inspect Runtime State

The container was inspected using `podman inspect`.

Important runtime properties were reviewed:

* Container status.
* Exit code.
* Error state.
* OOM status.
* Restart count.
* Published ports.
* Network mode.
* Runtime configuration.

The container was running with exit code `0` and no recorded runtime error.

## 4. Analyze Resource Usage

Container resource consumption was checked using:

```bash
podman stats --no-stream nginx-test
```

The observed CPU and memory consumption were low, and the container was not showing signs of resource exhaustion.

## 5. Debug from Inside the Container

An interactive shell was opened with:

```bash
podman exec -it nginx-test /bin/sh
```

Inside the container, running processes were examined using `ps aux`.

This confirmed that the Nginx master process and worker processes were active.

## 6. Review Application Configuration

The Nginx configuration file was examined:

```bash
cat /etc/nginx/nginx.conf
```

This provided visibility into:

* Worker configuration.
* Logging configuration.
* HTTP configuration.
* Included configuration files.

## 7. Validate Application Configuration

Nginx configuration syntax was tested with:

```bash
nginx -t
```

The test returned a successful syntax validation result.

## 8. Test Internal Connectivity

The Nginx service was tested from inside the container:

```bash
wget -qO- http://localhost
```

The default Nginx welcome page was returned successfully.

## 9. Verify External Connectivity

After exiting the container, the service was tested from the host:

```bash
curl -I http://localhost:8080
```

The response returned HTTP `200 OK`, confirming successful connectivity through the published port.

## 10. Troubleshooting Model

The lab followed a layered troubleshooting approach:

```text
Container Status
       ↓
Application Logs
       ↓
Runtime Inspection
       ↓
Resource Usage
       ↓
Process Inspection
       ↓
Configuration Validation
       ↓
Internal Connectivity
       ↓
External Connectivity
```

This approach helps isolate whether a problem originates from the container runtime, application process, configuration, resources, networking, or service availability.

## Result

The troubleshooting workflow confirmed that the Nginx container was operating normally and that both internal and external service connectivity were successful.
