# Lab 9: Using ENTRYPOINT and CMD — Methodology

## 1. Lab Preparation

The lab was performed on an Ubuntu system using Podman as the container engine. The Red Hat UBI 9 minimal image was used as the base image for all Containerfiles.

The lab was divided into four practical areas:

1. Exec-form `ENTRYPOINT` and `CMD`
2. Runtime `CMD` override
3. Script-based `ENTRYPOINT`
4. Shell-form `ENTRYPOINT` and `CMD`

---

## 2. Testing Exec-Form ENTRYPOINT and CMD

The first Containerfile defined:

```dockerfile
ENTRYPOINT ["echo", "Entrypoint says:"]
CMD ["Default CMD message"]
```

The image was built using:

```bash
podman build -t entrypoint-demo .
```

The container was first executed without additional arguments.

This verified the normal interaction between `ENTRYPOINT` and `CMD`.

The resulting output was:

```text
Entrypoint says: Default CMD message
```

A second test supplied a custom runtime argument:

```bash
podman run --rm entrypoint-demo "Custom message"
```

The resulting output was:

```text
Entrypoint says: Custom message
```

This demonstrated that the runtime argument replaced the default `CMD` value while the `ENTRYPOINT` remained unchanged.

---

## 3. Testing a Script-Based ENTRYPOINT

A shell script named `greet.sh` was created:

```sh
#!/bin/sh
echo "Welcome to $1 from $2"
```

The script was copied into the container image and configured as the `ENTRYPOINT`.

The image was built as:

```bash
podman build -t greet-demo .
```

The container was tested with runtime arguments:

```bash
podman run --rm greet-demo "Container Workshop" "Instructor"
```

The script processed the two arguments and produced:

```text
Welcome to Container Workshop from Instructor
```

This demonstrated how an entrypoint script can receive and process runtime parameters.

---

## 4. Testing ENTRYPOINT Override

The configured script entrypoint was replaced at runtime with `echo`:

```bash
podman run --rm \
  --entrypoint echo \
  greet-demo \
  "This completely replaces the ENTRYPOINT"
```

The resulting output was:

```text
This completely replaces the ENTRYPOINT
```

This confirmed that Podman can replace the image's configured `ENTRYPOINT` using the `--entrypoint` option.

---

## 5. Testing Shell-Form Instructions

The final Containerfile used shell-form syntax:

```dockerfile
ENTRYPOINT echo "Shell form ENTRYPOINT:"
CMD echo "Shell form CMD"
```

The image was built as:

```bash
podman build -t shell-form-demo .
```

The container was then executed.

The observed output was:

```text
Shell form ENTRYPOINT:
```

This test was compared with the earlier exec-form behavior to understand the difference between the two Containerfile instruction formats.

---

## 6. Validation Approach

Each major configuration was validated by:

1. Creating the Containerfile.
2. Building the corresponding image.
3. Running the container.
4. Supplying runtime arguments where applicable.
5. Recording the actual terminal output.
6. Capturing screenshots of the successful tests.

No expected output was used as a substitute for actual execution results.
