# Lab 6 — Command Reference

## 1. Create Project Directory

```bash
mkdir -p ~/Alrazzaq_Labs/Container/06-Building-Custom-Container-Images
cd ~/Alrazzaq_Labs/Container/06-Building-Custom-Container-Images
```

## 2. Create index.html

```bash
echo '<h1>Welcome to My Custom Nginx Container!</h1>' > index.html
```

Verify:

```bash
cat index.html
```

Expected:

```html
<h1>Welcome to My Custom Nginx Container!</h1>
```

## 3. Create Containerfile

```bash
cat > Containerfile <<'EOF'
# Use the official Nginx base image
FROM docker.io/nginx:alpine

# Set an environment variable
ENV AUTHOR="OpenShift Developer"

# Copy the custom HTML file to the Nginx web root
COPY index.html /usr/share/nginx/html

# Run a command to print a message
RUN echo "Container built by $AUTHOR" > /build-info.txt
EOF
```

Verify:

```bash
cat Containerfile
```

## 4. Build the Image

```bash
podman build -t my-custom-nginx .
```

## 5. List Images

```bash
podman images
```

The resulting custom image was:

```text
localhost/my-custom-nginx:latest
```

## 6. Inspect the Image

```bash
podman inspect localhost/my-custom-nginx
```

## 7. Verify Environment Variable

```bash
podman inspect --format '{{.Config.Env}}' localhost/my-custom-nginx
```

The custom variable was verified as:

```text
AUTHOR=OpenShift Developer
```

## 8. Verify Build-Time File

```bash
podman run --rm localhost/my-custom-nginx cat /build-info.txt
```

Expected:

```text
Container built by OpenShift Developer
```

## 9. Verify Copied HTML

```bash
podman run --rm localhost/my-custom-nginx cat /usr/share/nginx/html/index.html
```

Expected:

```html
<h1>Welcome to My Custom Nginx Container!</h1>
```

## 10. Run the Container

```bash
podman run -d --name custom-nginx -p 8080:80 localhost/my-custom-nginx
```

## 11. Verify Running Container

```bash
podman ps
```

## 12. Verify Port Mapping

```bash
podman port custom-nginx
```

Verified mapping:

```text
80/tcp -> 0.0.0.0:8080
```

## 13. Test Nginx

```bash
curl http://localhost:8080
```

Expected:

```html
<h1>Welcome to My Custom Nginx Container!</h1>
```

## 14. View Container Logs

```bash
podman logs custom-nginx
```

The logs confirmed successful Nginx startup and an HTTP `200` response.

## 15. Troubleshooting

List all containers:

```bash
podman ps -a
```

View logs:

```bash
podman logs <container_id>
```

Inspect the container:

```bash
podman inspect <container_id>
```

## 16. Screenshot Directory

```bash
ls -lh screenshots/
```

The lab evidence is stored in:

```text
screenshots/01-containerfile-and-html.png
screenshots/02-build-image.png
screenshots/03-image-inspection.png
screenshots/04-image-content-verification.png
screenshots/05-running-container.png
screenshots/06-web-test.png
screenshots/07-container-logs.png
```
