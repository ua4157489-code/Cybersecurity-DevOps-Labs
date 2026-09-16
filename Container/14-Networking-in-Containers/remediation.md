# Lab 14: Networking in Containers — Remediation and Best Practices

## 1. Minimize Published Ports

Only publish ports that are required by the application.

Instead of exposing unnecessary services:

```bash
-p 8080:80
```

should only be used when external host access is actually required.

---

## 2. Use Dedicated Networks

Applications that need to communicate with each other should use dedicated Podman networks.

Example:

```bash
podman network create lab-network
```

Then attach containers to the required network:

```bash
podman run -d \
  --network lab-network \
  ...
```

This helps separate application traffic from unrelated containers.

---

## 3. Review Network Configuration

Regularly inspect networks:

```bash
podman network ls
```

```bash
podman network inspect lab-network
```

This helps identify unexpected subnets, gateways, or configuration changes.

---

## 4. Verify Published Ports

Review container port mappings:

```bash
podman port webapp
```

Avoid exposing services unnecessarily to all host interfaces.

Where appropriate, bind services to a specific host address rather than exposing them broadly.

---

## 5. Monitor Container Connectivity

Use connectivity tests such as:

```bash
curl -I http://localhost:8080
```

For troubleshooting, combine connectivity testing with:

```bash
podman ps
podman port webapp
podman inspect webapp
```

---

## 6. Use Internal Networks When Appropriate

Applications that do not require external access can use isolated/internal network configurations where supported.

This reduces unnecessary exposure of backend services.

---

## 7. Apply Host Firewall Controls

Host firewall rules should restrict access to published container ports according to the application's requirements.

For example, review active firewall rules using the distribution's firewall tooling.

---

## 8. Document Network Architecture

Record:

* Network names
* Subnets
* Gateways
* Container IPs
* Published ports
* Required communication paths

This makes troubleshooting and security review easier.

---

## 9. Avoid Hard-Coding Dynamic Container IPs

Container IP addresses can change when containers are recreated.

In this lab, the container initially received one address and the recreated container received:

```text
10.89.0.3
```

For multi-container applications, prefer Podman network-based service discovery and container names where supported instead of relying on manually assigned dynamic IP addresses.

---

## 10. Recommended Troubleshooting Sequence

When investigating container networking problems:

```text
Check Container Status
        ↓
Check Published Ports
        ↓
Inspect Network
        ↓
Inspect Container Network Assignment
        ↓
Check Container IP/Gateway
        ↓
Test Application Connectivity
        ↓
Check Host Firewall
        ↓
Review Application Logs
```

This approach helps distinguish container runtime, network configuration, port publishing, firewall, and application-level connectivity problems.
