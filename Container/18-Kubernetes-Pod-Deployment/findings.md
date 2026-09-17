# Lab 18 — Findings

## 1. Kubernetes Cluster

A Kind Kubernetes cluster named `lab18` was successfully created.

The active context was:

```text
kind-lab18
```

The control-plane node was:

```text
lab18-control-plane
```

The node status was:

```text
Ready
```

The cluster was running Kubernetes `v1.34.0`.

## 2. Pod Deployment

The `nginx-pod` Pod was successfully created from the YAML manifest.

Observed state:

```text
READY:     1/1
STATUS:    Running
RESTARTS:  0
```

The Pod received the IP address:

```text
10.244.0.5
```

The Pod was scheduled on:

```text
lab18-control-plane
```

## 3. Container

The Pod contained one container:

```text
nginx-container
```

The container image was:

```text
nginx:latest
```

The resolved image was:

```text
docker.io/library/nginx@sha256:d0d674272be3be36f9a13d79194fa0db5aa630ab3ede9bec459d12f67370aaef
```

The running Nginx version was:

```text
1.31.6
```

## 4. Pod Lifecycle

Kubernetes reported the expected Pod lifecycle:

```text
Scheduled
Pulling
Pulled
Created
Started
```

No restart occurred during the verification period.

## 5. Nginx Configuration

The Nginx configuration test returned:

```text
syntax is ok
configuration file /etc/nginx/nginx.conf test is successful
```

This confirmed that the default Nginx configuration was syntactically valid.

## 6. HTTP Connectivity

An HTTP HEAD request from inside the container returned:

```text
HTTP/1.1 200 OK
```

The response included:

```text
Server: nginx/1.31.6
Content-Type: text/html
Content-Length: 896
```

The same HTTP `200 OK` response was obtained through Kubernetes port forwarding on local port `8080`.

The complete Nginx welcome page was also returned successfully.

## 7. Logs

Nginx startup logs confirmed:

* Configuration initialization completed.
* Nginx `1.31.6` started.
* Worker processes were created.
* An HTTP request was successfully served with status `200`.

## 8. Port Forwarding Observation

A later port-forward attempt failed because port `8080` was already in use.

The error was:

```text
bind: address already in use
```

The existing port-forward process was terminated, resolving the local port conflict.

## 9. Security and Operational Observations

The Pod is a basic demonstration workload and does not include production hardening.

The manifest does not specify:

* CPU or memory resource requests/limits
* Readiness or liveness probes
* Security context
* Non-root user configuration
* Image digest pinning
* Network policies

These controls should be considered before using a similar Pod definition in a production environment.
