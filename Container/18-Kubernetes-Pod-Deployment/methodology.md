# Lab 18 — Methodology

## Objective

The objective of this lab was to deploy and inspect a basic Kubernetes Pod running Nginx.

The exercise focused on understanding a Pod manifest, deploying the resource, examining its runtime state, reviewing logs and events, and validating network access.

## Environment Preparation

A Kubernetes client was already installed on the Ubuntu host. No Kubernetes cluster or context was initially configured.

Kind was installed and used to create a local Kubernetes cluster using Docker as the container backend.

The resulting cluster used:

* Kind `v0.30.0`
* Kubernetes `v1.34.0`
* Context `kind-lab18`
* Node `lab18-control-plane`

The node reached the `Ready` state before the Pod deployment began.

## Manifest Creation

A Pod manifest named `simple-pod.yaml` was created.

The manifest defined:

* Kubernetes API version `v1`
* Resource type `Pod`
* Pod name `nginx-pod`
* Label `app: nginx`
* Container name `nginx-container`
* Image `nginx:latest`
* Container port `80`

The manifest was validated using a client-side dry run before deployment.

## Pod Deployment

The Pod was deployed using:

```bash
kubectl apply -f simple-pod.yaml
```

The deployment returned:

```text
pod/nginx-pod created
```

The Pod was then monitored using:

```bash
kubectl get pods -o wide
```

The Pod reached:

```text
READY:     1/1
STATUS:    Running
RESTARTS:  0
```

The assigned Pod IP was `10.244.0.5`.

## Runtime Inspection

The complete Pod definition was examined using:

```bash
kubectl get pod nginx-pod -o yaml
```

Detailed runtime information was then obtained with:

```bash
kubectl describe pod nginx-pod
```

This provided information about scheduling, container state, networking, image information, and lifecycle events.

## Event Analysis

Kubernetes events were reviewed using:

```bash
kubectl get events --sort-by='.lastTimestamp'
```

The observed lifecycle consisted of:

1. Pod scheduled to `lab18-control-plane`
2. Nginx image pulled
3. Container created
4. Container started

No Pod failure or restart event was observed.

## Application Validation

Nginx logs were examined with:

```bash
kubectl logs nginx-pod
```

The logs confirmed Nginx `1.31.6` started successfully.

The Nginx configuration was then tested from inside the container:

```bash
kubectl exec nginx-pod -- nginx -t
```

The configuration syntax and configuration test both succeeded.

An internal HTTP request was performed against the Nginx listener:

```bash
kubectl exec nginx-pod -- curl -I http://localhost
```

The server returned HTTP `200 OK`.

## Port-Forward Testing

Kubernetes port forwarding was used to expose the Pod's port 80 on local port 8080:

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

The forwarded endpoint was tested from the Ubuntu host with:

```bash
curl -I http://127.0.0.1:8080
```

The request returned HTTP `200 OK` and the standard Nginx welcome page.

## Troubleshooting

During testing, a second port-forward attempt was made while port `8080` was already occupied.

The resulting error indicated:

```text
bind: address already in use
```

The issue was resolved by stopping the existing port-forward process.

This demonstrated the importance of checking local port availability when using `kubectl port-forward`.

## Evidence Collection

Five primary evidence screenshots were planned for the lab:

1. Pod deployment and status
2. Pod port forwarding and HTTP test
3. Pod inspection and logs
4. Nginx container verification
5. Kubernetes Pod events

All evidence should represent actual commands and results from the completed lab environment.
