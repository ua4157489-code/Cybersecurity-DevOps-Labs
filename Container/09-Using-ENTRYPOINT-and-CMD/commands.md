# Lab 9: Using ENTRYPOINT and CMD — Commands

## 1. Create Lab Directory

```bash
cd ~/Alrazzaq_Labs/Container
mkdir -p 09-Using-ENTRYPOINT-and-CMD/screenshots
cd 09-Using-ENTRYPOINT-and-CMD
```

---

## 2. Basic ENTRYPOINT and CMD

Create the Containerfile:

```bash
cat <<'EOF' > Containerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

ENTRYPOINT ["echo", "Entrypoint says:"]
CMD ["Default CMD message"]
EOF
```

Verify:

```bash
cat Containerfile
```

Build the image:

```bash
podman build -t entrypoint-demo .
```

Run using the default CMD:

```bash
podman run --rm entrypoint-demo
```

Expected/observed output:

```text
Entrypoint says: Default CMD message
```

Override the CMD:

```bash
podman run --rm entrypoint-demo "Custom message"
```

Observed output:

```text
Entrypoint says: Custom message
```

---

## 3. Script-Based ENTRYPOINT

Create the greeting script:

```bash
cat <<'EOF' > greet.sh
#!/bin/sh
echo "Welcome to $1 from $2"
EOF
```

Make the script executable:

```bash
chmod +x greet.sh
```

Create the Containerfile:

```bash
cat <<'EOF' > Containerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

COPY greet.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/greet.sh"]
CMD ["OpenShift Lab", "Red Hat"]
EOF
```

Verify:

```bash
cat greet.sh
cat Containerfile
```

Build:

```bash
podman build -t greet-demo .
```

Run with default arguments:

```bash
podman run --rm greet-demo
```

Run with custom arguments:

```bash
podman run --rm greet-demo "Container Workshop" "Instructor"
```

Observed custom output:

```text
Welcome to Container Workshop from Instructor
```

---

## 4. Override ENTRYPOINT

Replace the configured entrypoint with `echo`:

```bash
podman run --rm \
  --entrypoint echo \
  greet-demo \
  "This completely replaces the ENTRYPOINT"
```

Observed output:

```text
This completely replaces the ENTRYPOINT
```

---

## 5. Shell-Form ENTRYPOINT and CMD

Create the shell-form Containerfile:

```bash
cat <<'EOF' > Containerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

ENTRYPOINT echo "Shell form ENTRYPOINT:"
CMD echo "Shell form CMD"
EOF
```

Verify:

```bash
cat Containerfile
```

Build:

```bash
podman build -t shell-form-demo .
```

Run:

```bash
podman run --rm shell-form-demo
```

Observed output:

```text
Shell form ENTRYPOINT:
```

---

## 6. Verify Images

```bash
podman images | grep -E 'entrypoint-demo|greet-demo|shell-form-demo'
```

The lab produced the following local images:

```text
entrypoint-demo
greet-demo
shell-form-demo
```
