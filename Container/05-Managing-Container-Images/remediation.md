# Remediation & Best Practices — Container Images

---

## 1. Use Trusted Registries and Fully Qualified Names

Always state the registry explicitly so image resolution does not depend on the local
`registries.conf` search order:

```bash
podman pull docker.io/library/ubuntu:latest
```

Avoid pulling from unknown registries without verification. Where the organisation
operates a private registry or mirror, prefer it — it gives control over what enters
the environment and an audit trail of what was pulled.

---

## 2. Pin Images for Reproducibility

`latest` is a mutable pointer. For anything beyond experimentation, pin a release tag:

```bash
podman pull docker.io/library/ubuntu:20.04
```

For true immutability, pin the digest — this always resolves to the exact same content:

```bash
podman pull docker.io/library/ubuntu@sha256:513c074113a871b51a8d16ab445c88779d6452d937a164fb5cc479f32668a41d
```

**Addresses:** Finding 08.

---

## 3. Inspect Images Before Use

```bash
podman inspect docker.io/library/ubuntu:latest
```

Check at minimum:

- Image ID and digest
- OS and architecture (guard against pulling a mismatched platform)
- `Config.Env` — no credentials or unexpected variables
- `Config.Cmd` / `Entrypoint` — what actually runs
- Labels — declared version and maintainer
- Layer count and structure

**Addresses:** Findings 03, 04.

---

## 4. Scan Images for Vulnerabilities

Metadata inspection shows what the image *claims*; a scanner shows what it *contains*.
Scan before any production use:

```bash
trivy image docker.io/library/ubuntu:latest
grype docker.io/library/ubuntu:latest
```

Options include **Trivy**, **Grype** and **Clair**. The goal is to identify vulnerable
packages and outdated dependencies inside the image before deployment, and to re-scan
on a schedule as new CVEs are published against images already in production.

---

## 5. Minimise Image Size and Content

Use minimal base images where the workload permits and strip unnecessary packages.
Smaller images reduce:

- Attack surface
- Number of potentially vulnerable packages
- Storage consumption
- Pull and transfer time

---

## 6. Never Bake Secrets Into Layers

Because layers are immutable and stacked (Finding 05), deleting a file in a later layer
does **not** remove it from the earlier one — it remains extractable from the image.
Inject secrets at runtime via environment variables, mounted files or a secrets manager;
never via build-time `COPY` or `ENV`.

---

## 7. Establish an Image Lifecycle

```
Trusted Registry
       ↓
Image Pull (pinned tag or digest)
       ↓
Metadata Verification
       ↓
Vulnerability Scan
       ↓
Security / Functional Testing
       ↓
Approved Image (digest recorded)
       ↓
Deployment
       ↓
Scheduled Re-scan / Update
       ↓
Remove Deprecated Images
```

**Addresses:** Findings 03, 08.

---

## 8. Manage Local Image Storage Deliberately

Review what is stored and how much space it occupies:

```bash
podman images
podman system df
```

Remove a specific image when it is no longer required:

```bash
podman rmi docker.io/library/ubuntu:20.04
```

Clean dangling (untagged) images — the safer default:

```bash
podman image prune
```

Use the broader form with care, since it removes every image not currently referenced
by a container:

```bash
podman image prune -a
```

On build servers and shared hosts, prefer scoped removal or filtered pruning
(for example `--filter until=720h`) over a blanket `-a`.

**Addresses:** Finding 07.

---

## 9. Verify Identity by Digest, Not Tag

Record the digest of any approved image and verify it at deploy time. Tags can be
reassigned at the registry; digests cannot. Where the registry supports it, enable
signature verification (`podman image trust`, Sigstore/cosign) so unsigned or tampered
images are rejected before they run.

---

## 10. Document Filter and Tooling Behaviour

As Finding 01 showed, options such as `--filter=is-official=true` may silently return
nothing depending on Podman version and registry backend. Verify tooling behaviour in
the target environment rather than assuming parity with documentation or with Docker,
and record the observed result so later readers are not misled.
