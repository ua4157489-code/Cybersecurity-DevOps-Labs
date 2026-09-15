# Lab 6 — Remediation and Security Recommendations

## Overview

The custom Nginx container was successfully built and tested. No active security vulnerability was identified as part of this lab because the primary objective was container image construction.

The recommendations below describe security improvements that should be considered before using a custom container image in a production environment.

## 1. Use Trusted Base Images

The lab uses:

```dockerfile
FROM docker.io/nginx:alpine
```

Production images should be sourced from trusted registries and verified before use.

Recommended practices:

* Use official or organization-approved images.
* Verify image provenance.
* Pin images to controlled versions or digests where appropriate.
* Avoid untrusted third-party images.

## 2. Keep the Base Image Updated

Container images can contain operating-system packages and application dependencies with known vulnerabilities.

Recommended workflow:

```text
Pull updated base image
        ↓
Rebuild custom image
        ↓
Run vulnerability scan
        ↓
Test application
        ↓
Deploy approved image
```

Regular image rebuilding reduces exposure to known vulnerabilities.

## 3. Scan Container Images

Before production deployment, scan the custom image with a container security scanner.

Possible tools include:

* Trivy
* Grype
* Docker Scout
* Clair

Example using Trivy:

```bash
trivy image localhost/my-custom-nginx:latest
```

The scan results should be reviewed for critical and high-severity vulnerabilities.

## 4. Avoid Hardcoded Secrets

The Containerfile in this lab contains only a demonstration environment variable:

```dockerfile
ENV AUTHOR="OpenShift Developer"
```

Production Containerfiles should never contain:

* Passwords
* API keys
* Private keys
* Access tokens
* Cloud credentials
* Database credentials

Secrets should be provided through an appropriate secret-management mechanism at runtime.

## 5. Minimize Container Privileges

Containers should run with the minimum privileges required by the application.

Recommended controls include:

```bash
--read-only
--cap-drop=ALL
--security-opt=no-new-privileges
```

These options should be tested against application requirements before deployment.

## 6. Limit Network Exposure

The lab publishes:

```text
8080 -> 80
```

Only required application ports should be exposed in production.

Network access should also be restricted through:

* Cloud security groups.
* Host firewalls.
* Container network policies.
* Reverse proxies.
* Kubernetes network policies where applicable.

## 7. Use Immutable Image References

For production environments, image tags such as:

```text
latest
```

can change over time.

Where appropriate, production deployments should reference a specific version or immutable digest.

Example:

```text
nginx:alpine@sha256:<digest>
```

This improves reproducibility and supply-chain control.

## 8. Monitor Container Logs

The lab verified Nginx logs with:

```bash
podman logs custom-nginx
```

Production deployments should centralize container logs into an appropriate monitoring or SIEM platform.

Examples include:

* Wazuh
* Elastic Stack
* Splunk
* OpenSearch

Monitoring should detect unusual requests, errors, authentication failures, and other suspicious activity.

## 9. Rebuild After Security Updates

When the underlying Nginx or Alpine image receives security updates, the custom image should be rebuilt and retested.

Recommended lifecycle:

```text
Monitor advisories
      ↓
Update base image
      ↓
Rebuild image
      ↓
Scan image
      ↓
Test application
      ↓
Deploy
      ↓
Monitor
```

## 10. Production Security Checklist

Before deploying the image to production:

* [ ] Use a trusted base image.
* [ ] Pin the base image appropriately.
* [ ] Scan the image for vulnerabilities.
* [ ] Remove unnecessary packages.
* [ ] Avoid secrets in the image.
* [ ] Run with minimum privileges.
* [ ] Drop unnecessary Linux capabilities.
* [ ] Enable `no-new-privileges` where possible.
* [ ] Restrict network exposure.
* [ ] Centralize container logs.
* [ ] Monitor the running workload.
* [ ] Establish a regular image update process.
* [ ] Maintain image provenance and version tracking.

## Conclusion

The Lab 6 image successfully fulfilled its educational purpose. For production use, the image should undergo additional container-security controls including vulnerability scanning, trusted image verification, privilege reduction, network restriction, secret management, and continuous monitoring.

These controls help reduce container supply-chain risk and improve the security posture of containerized applications.
