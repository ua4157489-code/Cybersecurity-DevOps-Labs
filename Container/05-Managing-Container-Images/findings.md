# Findings — Managing Container Images

---

## Finding 01 — Image Search Works, Official Filter Does Not

**Observation.** `podman search ubuntu` returned results from Docker Hub. The variant
`podman search --filter=is-official=true ubuntu` produced no visible output.

**Evidence.** `screenshots/01-search-images.png`

**Assessment.** Search itself is available and usable for discovery. The official-image
filter should not be assumed to behave identically across Podman versions or registry
search backends — the filter is passed to the registry's search API, and support
varies. Do not treat an empty filtered result as proof that no official image exists.

**Severity.** Informational.

---

## Finding 02 — Multiple Tags Coexist Locally

**Observation.** Two tags of the same repository were stored simultaneously:

| Image | Size |
|---|---|
| `docker.io/library/ubuntu:latest` | ~112 MB |
| `docker.io/library/ubuntu:20.04` | ~75.2 MB |

**Evidence.** `screenshots/02-pulled-images.png`

**Assessment.** Tags are independent references within one repository, each resolving
to its own image ID. Multiple versions can be retained for comparison or rollback, at
the cost of disk usage. Shared layers reduce, but do not eliminate, that cost.

**Severity.** Informational.

---

## Finding 03 — Identity of the Current `latest` Image

**Observation.**

| Property | Value |
|---|---|
| Image ID | `e2e49769ecc7948a72b28e9748f2dc881a13f5cea02fc0faa99a7a5607e457f6` |
| Digest | `sha256:513c074113a871b51a8d16ab445c88779d6452d937a164fb5cc479f32668a41d` |
| Version label | Ubuntu 26.04 |
| Architecture | amd64 |
| OS | linux |
| Command | `/bin/bash` |
| Size | ~112 MB |
| Graph driver | overlay |

**Evidence.** `screenshots/03-image-metadata.png`

**Assessment.** The `latest` tag currently resolves to Ubuntu 26.04, but this is a
point-in-time observation. The tag is mutable; the digest is not. Any environment that
needs reproducibility must record and pin the digest rather than the tag.

**Severity.** Informational — with an operational risk noted (see Finding 08).

---

## Finding 04 — Image Environment Configuration

**Observation.**

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

**Evidence.** `screenshots/03-image-metadata.png`

**Assessment.** The image ships only the standard executable search path — no extra
environment variables were baked in. This is the expected minimal configuration for a
base image and confirms no unexpected variables (including accidental secrets) are
present in `Config.Env`. Inspecting `Config.Env` is a routine check, since environment
variables set at build time are visible to anyone who can pull the image.

**Severity.** Informational.

---

## Finding 05 — Layered Image Structure Confirmed

**Observation.** The image contains two RootFS layers:

```
sha256:4a6e4a6c5956212bf74c556250eb0028096f13d74501e43af51a1dbed8d69749
sha256:4f41b99e67b8aeaedacb31ca888054a4b877577371f0bbe01196e136d2793635
```

**Evidence.** `screenshots/04-image-layers.png`

**Assessment.** The image is an ordered stack of content-addressed, read-only layers
union-mounted by the overlay driver. Two practical consequences follow: layers are
deduplicated and cached across images, so subsequent pulls transfer less data; and a
file deleted in a later layer still exists in the earlier one, so credentials written
into any layer remain recoverable from the image regardless of later deletion.

**Severity.** Informational — with a security implication for image builders.

---

## Finding 06 — Targeted Removal Behaves as Expected

**Observation.** `podman rmi docker.io/library/ubuntu:20.04` removed only that image;
`ubuntu:latest` remained.

**Evidence.** `screenshots/05-remove-image.png`

**Assessment.** Removal is scoped to a single image reference, not the repository.
Podman will also refuse to delete an image referenced by an existing container unless
`-f` is used — a safeguard that should be respected rather than routinely bypassed.

**Severity.** Informational.

---

## Finding 07 — Prune Clears the Image Store

**Observation.** `podman image prune -a` removed the remaining image after interactive
confirmation. `podman images` and `podman image ls` both returned empty tables.

**Evidence.** `screenshots/06-image-cleanup.png`

**Assessment.** `-a` broadens pruning from dangling images to every image not in use by
a container. This is effective for reclaiming disk space but destructive on shared or
build hosts, where "not currently used by a container" does not mean "not needed".

**Severity.** Low — operational risk if run unscoped in shared environments.

---

## Finding 08 — `latest` Is Not a Version Identifier

**Observation.** The tag `latest` currently maps to Ubuntu 26.04 and to the digest
recorded in Finding 03. Nothing about the tag itself guarantees that mapping over time.

**Assessment.** Deployments that reference `latest` can silently change base OS release
between pulls, producing environment drift and non-reproducible builds. This is the
most operationally significant issue surfaced by the lab.

**Severity.** Medium in a production context; not applicable to this test environment.

**Remediation.** See `remediation.md`, sections 2 and 7.

---

## Overall Result

The full lifecycle was demonstrated end to end:

```
Search → Pull → List → Inspect Metadata → Inspect Layers → Remove → Prune → Empty Store
```

All stages completed successfully. The only unexpected behaviour was the empty result
from the official-image search filter (Finding 01).
