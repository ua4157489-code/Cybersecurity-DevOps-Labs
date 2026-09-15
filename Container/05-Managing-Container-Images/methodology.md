# Methodology — Managing Container Images

## 1. Scope

This lab covers the local lifecycle of container images under Podman: discovery,
acquisition, inspection, structural analysis, and removal. It does not cover running
containers, building images, or registry authentication.

## 2. Environment Preparation

Podman was verified before any image operation:

```bash
podman --version
```

Reported version: **Podman 4.9.3**, running rootless on Linux/Ubuntu with the
**overlay** storage driver and **Docker Hub** as the configured registry.

## 3. Image Discovery

Images were located through the registry search interface:

```bash
podman search ubuntu
```

A filtered variant was tested to restrict results to official images:

```bash
podman search --filter=is-official=true ubuntu
```

The unfiltered query returned results; the filtered query returned no visible output.
The filter behaviour was recorded as observed rather than assumed, since search
filtering depends on both the Podman version and the registry's search API.

## 4. Image Acquisition

Two tags from the same repository were pulled using fully qualified names so that
registry resolution was unambiguous:

```bash
podman pull docker.io/library/ubuntu:latest
podman pull docker.io/library/ubuntu:20.04
```

The local store was then enumerated:

```bash
podman images
```

Observed sizes: `ubuntu:latest` ≈ 112 MB, `ubuntu:20.04` ≈ 75.2 MB.

## 5. Metadata Inspection

Full metadata was retrieved for the current release:

```bash
podman inspect docker.io/library/ubuntu:latest
```

Observed values of interest:

- **Image ID:** `e2e49769ecc7948a72b28e9748f2dc881a13f5cea02fc0faa99a7a5607e457f6`
- **Digest:** `sha256:513c074113a871b51a8d16ab445c88779d6452d937a164fb5cc479f32668a41d`
- **Architecture:** amd64
- **OS:** linux
- **Size:** ≈ 112 MB
- **Command:** `/bin/bash`
- **Storage driver:** overlay
- **Version label:** Ubuntu 26.04

A single field was then extracted with a Go template to demonstrate targeted querying:

```bash
podman inspect --format "{{.Config.Env}}" docker.io/library/ubuntu:latest
```

Result:

```
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
```

## 6. Layer Analysis

The RootFS layer digests were extracted and formatted:

```bash
podman image inspect docker.io/library/ubuntu:latest \
  --format '{{json .RootFS.Layers}}' | jq
```

Two layer digests were observed. This confirms that a container image is composed of
stacked filesystem layers rather than a single undifferentiated filesystem object, and
that the overlay driver union-mounts those layers to present one root filesystem.

## 7. Targeted Image Removal

A single image was removed to demonstrate scoped deletion:

```bash
podman rmi docker.io/library/ubuntu:20.04
```

`ubuntu:latest` remained available afterwards, confirming that removal is
image-specific rather than repository-wide.

## 8. Image Cleanup

All remaining unreferenced images were removed:

```bash
podman image prune -a
```

The operation required interactive confirmation. Final verification:

```bash
podman images
podman image ls
```

Both returned empty image tables.

## 9. Evidence Collection

Six screenshots were captured, one per workflow stage:

| # | File | Stage |
|---|---|---|
| 01 | `01-search-images.png` | Image searching |
| 02 | `02-pulled-images.png` | Image pulling and listing |
| 03 | `03-image-metadata.png` | Metadata inspection |
| 04 | `04-image-layers.png` | Layer inspection |
| 05 | `05-remove-image.png` | Individual image removal |
| 06 | `06-image-cleanup.png` | Prune and final verification |

Each screenshot corresponds directly to the documented step above and is embedded with
an explanation in `README.md`.

## 10. Limitations

- Findings reflect a single host, a single Podman version and a single registry.
- `latest` is mutable; the recorded ID, digest and version label are point-in-time.
- No vulnerability scanning or runtime testing was performed in this lab.
