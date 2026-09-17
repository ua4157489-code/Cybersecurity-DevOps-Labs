# Lab 20 — Methodology

## 1. Purpose

The purpose of this lab was to demonstrate how Kubernetes exposes applications using different networking mechanisms.

The practical implementation focused on:

1. Deploying an Nginx application.
2. Exposing Nginx through ClusterIP.
3. Exposing Nginx through NodePort.
4. Installing the Ingress-NGINX controller.
5. Creating a hostname-based Ingress.
6. Testing each access path.

The lab was performed using Kubernetes with Kind.

---

## 2. Environment Preparation

A fresh Ubuntu environment was used to create a Kind Kubernetes cluster.

The cluster was created with:

```text
Kind:       v0.30.0
Kubernetes: v1.34.0
kubectl:    v1.36.2
```

The resulting cluster contained one control-plane node:

```text
lab20-control-plane
```

The node reached the `Ready` state.

Its Kubernetes internal IP was:

```text
172.18.0.2
```

The node used:

```text
containerd://2.1.3
```

as its container runtime.

---

## 3. Application Deployment

An Nginx Deployment was created using the official Nginx image:

```bash
kubectl create deployment nginx --image=nginx:latest
```

The Deployment successfully reached:

```text
1/1 Ready
1 Up-to-Date
1 Available
```

The resulting pod was:

```text
nginx-7c5d8bf9f7-8cb5z
```

with IP:

```text
10.244.0.5
```

Nginx was verified as version:

```text
1.31.6
```

---

## 4. ClusterIP Implementation

The first service exposure method was ClusterIP.

The Deployment was exposed using:

```bash
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP \
  --name=nginx-clusterip
```

The resulting service received:

```text
ClusterIP: 10.96.210.14
Port:       80/TCP
```

The Service selected the Nginx pod through:

```text
app=nginx
```

The endpoint was:

```text
10.244.0.5:80
```

A temporary curl pod was used to test internal service connectivity.

The test returned:

```text
HTTP/1.1 200 OK
```

The Nginx welcome page was also successfully retrieved.

This demonstrated successful internal service discovery and routing.

---

## 5. NodePort Implementation

The second exposure method was NodePort.

The Deployment was exposed using:

```bash
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=NodePort \
  --name=nginx-nodeport
```

The resulting service used:

```text
ClusterIP: 10.96.229.77
NodePort:  31787
```

The application endpoint remained:

```text
10.244.0.5:80
```

Because the cluster was running inside a Kind container, the NodePort was tested from the Kind control-plane container.

The request to:

```text
172.18.0.2:31787
```

returned:

```text
HTTP/1.1 200 OK
```

The Nginx welcome page was successfully returned.

---

## 6. Ingress Controller Installation

Before installation, the cluster had no IngressClass or Ingress controller.

The Kind-compatible Ingress-NGINX deployment was installed.

The controller reached:

```text
1/1 Running
```

The controller pod was:

```text
ingress-nginx-controller-f5784567-gpmd8
```

with IP:

```text
10.244.0.10
```

The installation created the `nginx` IngressClass.

The controller Service exposed:

```text
HTTP: 31089
HTTPS: 31692
```

The Service was a LoadBalancer type, but its external IP remained:

```text
<pending>
```

This was expected in the Kind environment.

---

## 7. Ingress Configuration

An Ingress resource named:

```text
nginx-ingress
```

was created.

The Ingress used:

```text
IngressClass: nginx
Host:         nginx.example.com
Path:         /
```

The backend was:

```text
nginx-clusterip:80
```

The controller successfully synchronized the resource.

The Ingress reported:

```text
Address: localhost
```

and an event showed:

```text
Scheduled for sync
```

---

## 8. Ingress Validation

The Ingress was tested through the HTTP NodePort of the Ingress-NGINX controller.

The request used:

```text
Host: nginx.example.com
```

and was sent to:

```text
127.0.0.1:31089
```

The controller returned:

```text
HTTP/1.1 200 OK
```

The same successful result was obtained using the Kind node IP:

```text
172.18.0.2:31089
```

The response contained the standard Nginx welcome page.

---

## 9. Validation Approach

Each exposure mechanism was validated independently.

### ClusterIP

```text
curl → nginx-clusterip → nginx pod
```

Result:

```text
HTTP 200
```

### NodePort

```text
curl → Kind node:31787 → nginx-nodeport → nginx pod
```

Result:

```text
HTTP 200
```

### Ingress

```text
curl + Host header
        ↓
Ingress-NGINX :31089
        ↓
nginx-ingress
        ↓
nginx-clusterip:80
        ↓
nginx pod
```

Result:

```text
HTTP 200
```

---

## 10. Evidence Collection

Five screenshots were collected to document the lab:

```text
01-cluster-and-nginx-deployment.png
02-clusterip-service-and-test.png
03-nodeport-service-and-test.png
04-ingress-controller-and-resource.png
05-ingress-http-test.png
```

The screenshots document the cluster state, Services, Ingress controller, Ingress resource, and successful HTTP testing.

---

## 11. Completion Criteria

The lab was considered successful when:

* Kubernetes node reached Ready state.
* Nginx Deployment reached 1/1 available.
* ClusterIP Service had a valid endpoint.
* ClusterIP returned HTTP 200.
* NodePort returned HTTP 200.
* Ingress-NGINX controller reached Running state.
* `nginx` IngressClass was available.
* Ingress resource successfully synchronized.
* Host-based Ingress routing returned HTTP 200.

All listed criteria were successfully demonstrated during the lab.
