# Commands — Managing Container Images

All commands executed in this lab, in order, with their purpose and observed result.

---

## 1. Environment Check

```bash
podman --version
```

Purpose: confirm the container engine and version before testing behaviour that varies
between releases.
Result: `podman version 4.9.3`.

---

## 2. Image Search

```bash
podman search ubuntu
```

Purpose: query configured registries for repositories matching `ubuntu`.
Result: a table of Docker Hub repositories with names and descriptions.

```bash
podman search --filter=is-official=true ubuntu
```

Purpose: restrict results to official images only.
Result: no visible output in this environment.

---

## 3. Image Pull

```bash
podman pull docker.io/library/ubuntu:latest
podman pull docker.io/library/ubuntu:20.04
```

Purpose: download two tags of the same repository using fully qualified names.
Result: both pulls completed; blobs downloaded and manifests written.

---

## 4. List Local Images

```bash
podman images
podman image ls        # equivalent alias
```

Purpose: enumerate the local image store.
Result:

| Repository | Tag | Size |
|---|---|---|
| `docker.io/library/ubuntu` | latest | ~112 MB |
| `docker.io/library/ubuntu` | 20.04 | ~75.2 MB |

---

## 5. Inspect Metadata

```bash
podman inspect docker.io/library/ubuntu:latest
```

Purpose: retrieve the full JSON configuration and identity of the image.
Result (selected fields):

```
Id           e2e49769ecc7948a72b28e9748f2dc881a13f5cea02fc0faa99a7a5607e457f6
Digest       sha256:513c074113a871b51a8d16ab445c88779d6452d937a164fb5cc479f32668a41d
Architecture amd64
Os           linux
Cmd          /bin/bash
GraphDriver  overlay
Label        version=26.04
```

---

## 6. Extract a Single Field

```bash
podman inspect --format "{{.Config.Env}}" docker.io/library/ubuntu:latest
```

Purpose: pull one value out of the inspect document using a Go template.
Result:

```
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
```

---

## 7. Inspect Layers

```bash
podman image inspect docker.io/library/ubuntu:latest \
  --format '{{json .RootFS.Layers}}' | jq
```

Purpose: list the filesystem layer digests that compose the image.
Result:

```json
[
  "sha256:4a6e4a6c5956212bf74c556250eb0028096f13d74501e43af51a1dbed8d69749",
  "sha256:4f41b99e67b8aeaedacb31ca888054a4b877577371f0bbe01196e136d2793635"
]
```

---

## 8. Remove a Specific Image

```bash
podman rmi docker.io/library/ubuntu:20.04
```

Purpose: delete one image without affecting other tags.
Result: `ubuntu:20.04` untagged and deleted; `ubuntu:latest` retained.

---

## 9. Prune Unused Images

```bash
podman image prune -a
```

Purpose: remove all images not referenced by any container (`-a` extends the scope
beyond dangling images).
Result: confirmation prompt accepted; remaining image removed.

---

## 10. Final Verification

```bash
podman images
podman image ls
```

Purpose: confirm the image store is empty.
Result: empty image table from both commands.

---

## Quick Reference

| Task | Command |
|---|---|
| Show version | `podman --version` |
| Search registry | `podman search <term>` |
| Filter to official | `podman search --filter=is-official=true <term>` |
| Pull image | `podman pull <registry>/<repo>:<tag>` |
| List images | `podman images` / `podman image ls` |
| Full metadata | `podman inspect <image>` |
| Single field | `podman inspect --format "{{.Field}}" <image>` |
| Layer digests | `podman image inspect <image> --format '{{json .RootFS.Layers}}' \| jq` |
| Image history | `podman history <image>` |
| Remove image | `podman rmi <image>` |
| Force remove | `podman rmi -f <image>` |
| Prune dangling | `podman image prune` |
| Prune all unused | `podman image prune -a` |
| Disk usage | `podman system df` |
