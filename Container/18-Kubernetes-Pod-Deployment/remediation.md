# Lab 18 — Remediation and Improvements

## 1. Pin Container Images

The lab uses:

```yaml
image: nginx:latest
```

For production workloads, avoid mutable tags such as `latest`.

A specific version or immutable image digest should be used to improve deployment consistency and supply-chain control.

Example:

```yaml
image: nginx:1.31.6
```

For stronger reproducibility, an image digest can be used.

## 2. Define Resource Requests and Limits

The Pod currently has no resource requests or limits.

Production workloads should define appropriate CPU and memory values.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

This helps Kubernetes schedule workloads predictably and prevents unrestricted resource consumption.

## 3. Add Health Probes

The Pod does not currently define readiness or liveness probes.

A readiness probe can prevent traffic from being sent to an application before it is ready.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

A liveness probe can help Kubernetes detect an unhealthy application.

## 4. Configure Security Context

The lab Pod uses the default container security configuration.

Production workloads should consider an appropriate security context, including:

* Running as a non-root user where supported
* Dropping unnecessary Linux capabilities
* Preventing privilege escalation
* Using a read-only root filesystem where practical

## 5. Use a Deployment for Long-Running Applications

A standalone Pod is useful for learning and testing, but production applications normally use a higher-level controller such as a Deployment.

A Deployment provides:

* Replica management
* Rolling updates
* Self-healing
* Declarative application versioning

## 6. Apply Network Controls

The lab does not define a NetworkPolicy.

In environments where network isolation is required, NetworkPolicies should be used to restrict which workloads can communicate with the Pod.

## 7. Monitor Pod Events and Logs

The lab demonstrated the usefulness of:

```bash
kubectl describe pod nginx-pod
kubectl get events --sort-by='.lastTimestamp'
kubectl logs nginx-pod
```

These commands should be part of the initial troubleshooting workflow when a Pod enters states such as:

```text
Pending
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
```

## 8. Avoid Local Port Conflicts

The port-forward test demonstrated that local port `8080` cannot be reused while another process is listening on it.

Before starting a port-forward, check the port if necessary:

```bash
ss -ltnp | grep ':8080'
```

An alternative local port can also be selected:

```bash
kubectl port-forward pod/nginx-pod 8081:80
```

## Conclusion

The basic Pod deployment worked successfully. The main improvements concern production hardening rather than fixing a deployment failure.

The next logical Kubernetes labs are:

* Multi-container Pods
* Deployments and ReplicaSets
* Services
* ConfigMaps and Secrets
* Health probes
* Persistent Volumes
* Kubernetes security contexts
* NetworkPolicies
