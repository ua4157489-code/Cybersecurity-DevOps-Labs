# Lab 20 — Remediation and Security Recommendations

## Overview

The Lab 20 implementation was functionally successful. No critical application exposure failure was identified during testing.

The recommendations below focus on improving security and production readiness when applying the same Kubernetes concepts outside the isolated lab environment.

---

## 1. Prefer ClusterIP for Internal-Only Applications

If an application does not need external access, use a ClusterIP Service.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### Recommendation

Avoid exposing internal applications through NodePort when direct external access is unnecessary.

---

## 2. Restrict NodePort Access

The lab used:

```text
NodePort: 31787
```

NodePort can make a Service reachable through the node network.

### Recommendation

If NodePort is required:

* Restrict access using cloud security groups.
* Restrict access using host firewalls.
* Limit access to trusted networks.
* Avoid exposing unnecessary NodePorts.
* Monitor access to exposed node ports.

For AWS environments, security-group rules should permit only required source networks.

---

## 3. Use Ingress for HTTP/HTTPS Routing

Ingress provides centralized host and path-based routing.

The lab used:

```text
nginx.example.com
```

with:

```text
/
```

routed to:

```text
nginx-clusterip:80
```

### Recommendation

For production applications with multiple HTTP services, use an Ingress controller or Gateway API implementation instead of creating many externally exposed NodePorts.

---

## 4. Enable TLS

The lab validated HTTP only.

Production applications should normally use HTTPS.

A production Ingress can be configured with a TLS secret:

```yaml
spec:
  tls:
  - hosts:
    - nginx.example.com
    secretName: nginx-tls
```

### Recommendation

Use trusted certificates and redirect HTTP traffic to HTTPS where appropriate.

Certificate automation can be implemented with an approved certificate-management solution such as cert-manager.

---

## 5. Apply Network Policies

The lab did not implement NetworkPolicy restrictions.

Network policies can restrict which workloads are allowed to communicate with one another.

Example concept:

```text
Ingress Controller
        |
        v
   Nginx Service
        |
        v
   Nginx Pod
```

### Recommendation

Implement NetworkPolicies that allow only required application flows.

For example:

* Permit ingress traffic from the Ingress controller to Nginx.
* Restrict unnecessary pod-to-pod communication.
* Deny unnecessary lateral traffic.

NetworkPolicy support depends on the Kubernetes networking implementation being used.

---

## 6. Use Least-Privilege RBAC

The lab used Kubernetes commands to create and inspect resources.

Production environments should avoid unnecessary administrative permissions.

### Recommendation

Use:

* Dedicated ServiceAccounts.
* Role and RoleBinding resources.
* ClusterRole only where required.
* Least-privilege permissions.
* Regular RBAC reviews.

Avoid using cluster-admin permissions for routine application workloads.

---

## 7. Avoid Hard-Coded Sensitive Configuration

The lab used the public Nginx image and did not require application secrets.

For applications requiring credentials:

### Recommendation

Do not place passwords, API keys, or tokens directly inside manifests.

Use Kubernetes Secrets together with appropriate access controls.

For higher-security environments, consider an external secrets-management system.

---

## 8. Pin Container Images

The lab used:

```text
nginx:latest
```

The actual deployed version was:

```text
nginx/1.31.6
```

Using `latest` can cause future deployments to retrieve a different image version.

### Recommendation

Production deployments should use a controlled version tag or, preferably, an immutable image digest.

Example:

```yaml
image: nginx:<approved-version>
```

or:

```yaml
image: nginx@sha256:<approved-digest>
```

This improves deployment reproducibility and supply-chain control.

---

## 9. Scan Container Images

The lab did not include container image vulnerability scanning.

### Recommendation

Integrate image scanning into the development and deployment process.

The workflow can include:

```text
Build
  ↓
Image Scan
  ↓
Policy Check
  ↓
Registry
  ↓
Deployment
```

Only approved images should be deployed to production environments.

---

## 10. Monitor Ingress Traffic

Ingress controllers process application HTTP traffic and should be monitored.

### Recommendation

Collect:

* HTTP status codes.
* Request rates.
* Client addresses where appropriate.
* Error rates.
* Suspicious paths.
* Authentication failures.
* Excessive request rates.

Ingress logs can be integrated into a SIEM or centralized logging platform.

---

## 11. Protect Against Excessive Requests

Public HTTP endpoints may be targeted by automated traffic.

### Recommendation

Where appropriate, implement:

* Rate limiting.
* Request-size limits.
* Connection limits.
* Web Application Firewall controls.
* Authentication and authorization.
* DDoS protection.

The exact controls should be selected according to the application's requirements and deployment environment.

---

## 12. Use EndpointSlice for Modern Kubernetes Integrations

The lab displayed:

```text
Warning: v1 Endpoints is deprecated in v1.33+;
use discovery.k8s.io/v1 EndpointSlice
```

### Recommendation

For custom automation and Kubernetes integrations, use the modern EndpointSlice API:

```text
discovery.k8s.io/v1
```

rather than depending on the deprecated `v1 Endpoints` API.

The warning did not affect the successful Service tests performed in this lab.

---

## 13. Production LoadBalancer Consideration

The Kind environment reported:

```text
EXTERNAL-IP: <pending>
```

for the Ingress controller Service.

This is expected because Kind does not automatically provide the same cloud LoadBalancer integration as a managed Kubernetes service.

### Recommendation

In a production cloud environment, use the appropriate load-balancing integration provided by the Kubernetes platform.

For AWS-based Kubernetes deployments, the ingress/load-balancing architecture should be designed according to the organization's networking and security requirements.

---

## 14. Logging and Monitoring

The lab verified application availability through direct HTTP tests.

Production deployments should additionally monitor:

```text
Application
    ↓
Service
    ↓
Ingress
    ↓
Load Balancer
    ↓
Network
```

### Recommendation

Centralize logs and metrics and create alerts for:

* HTTP 4xx spikes.
* HTTP 5xx spikes.
* Ingress controller failures.
* Pod restarts.
* Deployment availability problems.
* Unusual traffic patterns.

---

## 15. Final Security Posture

The completed lab successfully demonstrated Kubernetes service exposure and Ingress routing.

For a production implementation, the following controls should be considered:

```text
                  Internet
                     |
              Load Balancer
                     |
              Ingress + TLS
                     |
             Authentication
                     |
                Service
                     |
              NetworkPolicy
                     |
                   Pod
                     |
             Logging/Monitoring
```

The lab itself remained a controlled Kubernetes environment, while these recommendations describe additional controls appropriate for production deployments.

---

## Conclusion

No critical functional issue was identified during Lab 20 testing.

The main production-hardening recommendations are:

1. Use ClusterIP when external exposure is unnecessary.
2. Restrict NodePort access when NodePort is required.
3. Use Ingress for centralized HTTP/HTTPS routing.
4. Enable TLS for production HTTP applications.
5. Apply NetworkPolicies.
6. Use least-privilege RBAC.
7. Pin container images instead of relying on `latest`.
8. Scan container images before deployment.
9. Monitor and centralize Ingress/application logs.
10. Use EndpointSlice for modern Kubernetes integrations.
