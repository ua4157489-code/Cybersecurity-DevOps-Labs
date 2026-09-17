# Lab 20 — Findings

## 1. Summary

Lab 20 successfully demonstrated Kubernetes application exposure using ClusterIP, NodePort, and Ingress-NGINX.

The Nginx application was successfully deployed and reached through all three tested exposure mechanisms.

---

## 2. Application Finding

### Finding F-01 — Nginx Deployment Successfully Available

**Status:** Verified

The Nginx Deployment reached:

```text
READY:       1/1
UP-TO-DATE:  1
AVAILABLE:   1
```

The running pod was:

```text
nginx-7c5d8bf9f7-8cb5z
```

Pod IP:

```text
10.244.0.5
```

Nginx version:

```text
1.31.6
```

No pod restart was observed during validation.

---

## 3. ClusterIP Finding

### Finding F-02 — Internal ClusterIP Access Successfully Verified

**Status:** Verified

The ClusterIP Service was:

```text
nginx-clusterip
```

Configuration:

```text
Type:       ClusterIP
ClusterIP:  10.96.210.14
Port:       80/TCP
Endpoint:   10.244.0.5:80
```

An internal curl test returned:

```text
HTTP/1.1 200 OK
```

The standard Nginx response page was successfully returned.

This confirms that the Kubernetes Service selector correctly mapped the Service to the Nginx pod.

---

## 4. NodePort Finding

### Finding F-03 — NodePort Access Successfully Verified

**Status:** Verified

The NodePort Service was:

```text
nginx-nodeport
```

Configuration:

```text
Type:       NodePort
ClusterIP:  10.96.229.77
Port:       80/TCP
NodePort:   31787
Endpoint:   10.244.0.5:80
```

The Kind node IP was:

```text
172.18.0.2
```

A request to:

```text
172.18.0.2:31787
```

returned:

```text
HTTP/1.1 200 OK
```

The Nginx welcome page was returned successfully.

---

## 5. Ingress Controller Finding

### Finding F-04 — Ingress-NGINX Controller Successfully Running

**Status:** Verified

The controller pod was:

```text
ingress-nginx-controller-f5784567-gpmd8
```

Status:

```text
1/1 Running
```

Pod IP:

```text
10.244.0.10
```

The available IngressClass was:

```text
nginx
```

The controller Service exposed:

```text
HTTP: 31089
HTTPS: 31692
```

---

## 6. Ingress Routing Finding

### Finding F-05 — Host-Based Ingress Routing Successfully Verified

**Status:** Verified

The Ingress resource:

```text
nginx-ingress
```

was configured for:

```text
Host: nginx.example.com
Path: /
```

Backend:

```text
nginx-clusterip:80
```

The Ingress controller successfully synchronized the configuration.

Testing with:

```text
Host: nginx.example.com
```

returned:

```text
HTTP/1.1 200 OK
```

The returned content was the Nginx welcome page.

This demonstrates successful routing from the Ingress controller to the ClusterIP Service and then to the Nginx pod.

---

## 7. Environment Finding

### Finding F-06 — Kind LoadBalancer External IP Remained Pending

**Status:** Expected

The Ingress controller Service was:

```text
Type: LoadBalancer
External-IP: <pending>
```

This occurred because the lab used a local Kind Kubernetes environment rather than a Kubernetes cluster with a cloud LoadBalancer implementation.

The Ingress remained functional through the Kind-compatible NodePort exposure.

---

## 8. Kubernetes API Warning

### Finding F-07 — Legacy Endpoints API Warning

During verification, Kubernetes displayed:

```text
Warning: v1 Endpoints is deprecated in v1.33+;
use discovery.k8s.io/v1 EndpointSlice
```

This warning did not prevent the Service from functioning.

The Service successfully reported:

```text
10.244.0.5:80
```

and the ClusterIP test returned HTTP 200.

The warning indicates that EndpointSlice is the preferred API for newer Kubernetes versions.

---

## 9. Security Observations

### Observation 1 — ClusterIP Limits Direct Exposure

The ClusterIP Service provides internal cluster access rather than directly exposing the application through a public external address.

### Observation 2 — NodePort Expands Network Exposure

NodePort makes the application reachable through a port on the Kubernetes node. In production environments, firewall rules and network policies should restrict unnecessary access.

### Observation 3 — Ingress Centralizes HTTP Routing

Ingress provides a centralized mechanism for host/path-based HTTP routing and can be used as a point for TLS termination and additional HTTP security controls.

### Observation 4 — TLS Was Not Configured

This lab validated HTTP routing only.

The Ingress used:

```text
Port: 80
```

HTTPS was available from the controller Service through port `31692`, but HTTPS/TLS configuration was not part of the completed lab validation.

---

## 10. Overall Result

The lab successfully achieved its functional objectives.

Verified:

```text
ClusterIP  → HTTP 200
NodePort   → HTTP 200
Ingress    → HTTP 200
```

No application availability failure was observed during the final verification.
