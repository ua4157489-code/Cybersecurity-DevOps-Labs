# Lab 6 — Findings

## Executive Summary

The custom Nginx container image was successfully built and deployed using Podman on an Ubuntu AWS EC2 instance.

All required Containerfile instructions were successfully demonstrated, and the resulting container served the custom HTML page through Nginx.

## Finding 1 — Custom Image Successfully Built

**Status:** Verified

The following command successfully created the custom image:

```bash
podman build -t my-custom-nginx .
```

The resulting image was available as:

```text
localhost/my-custom-nginx:latest
```

Verified image ID:

```text
08e57ffc4b4b
```

## Finding 2 — Base Image Successfully Used

**Status:** Verified

The Containerfile used:

```dockerfile
FROM docker.io/nginx:alpine
```

Image inspection confirmed that the custom image was derived from the Nginx Alpine base image.

## Finding 3 — Environment Variable Successfully Configured

**Status:** Verified

The Containerfile defined:

```dockerfile
ENV AUTHOR="OpenShift Developer"
```

The environment configuration was verified from the resulting image:

```text
AUTHOR=OpenShift Developer
```

## Finding 4 — Build-Time Command Successfully Executed

**Status:** Verified

The Containerfile contained:

```dockerfile
RUN echo "Container built by $AUTHOR" > /build-info.txt
```

The resulting file was tested from a temporary container:

```bash
podman run --rm localhost/my-custom-nginx cat /build-info.txt
```

Verified output:

```text
Container built by OpenShift Developer
```

This confirms that the `RUN` instruction was executed successfully during image construction.

## Finding 5 — Custom HTML Successfully Copied

**Status:** Verified

The Containerfile copied the custom page using:

```dockerfile
COPY index.html /usr/share/nginx/html
```

The resulting content was verified inside the image:

```html
<h1>Welcome to My Custom Nginx Container!</h1>
```

## Finding 6 — Container Successfully Deployed

**Status:** Verified

The container was started using:

```bash
podman run -d --name custom-nginx -p 8080:80 localhost/my-custom-nginx
```

The container successfully exposed Nginx through host port `8080`.

Verified mapping:

```text
80/tcp -> 0.0.0.0:8080
```

## Finding 7 — Web Application Successfully Served

**Status:** Verified

The application was tested with:

```bash
curl http://localhost:8080
```

The server returned:

```html
<h1>Welcome to My Custom Nginx Container!</h1>
```

This confirmed successful communication between the host and the Nginx service running inside the container.

## Finding 8 — HTTP Request Successfully Logged

**Status:** Verified

The Nginx logs recorded:

```text
"GET / HTTP/1.1" 200
```

The HTTP `200` response confirms that the request was successfully processed.

## Security Observations

This lab was primarily focused on container image construction rather than vulnerability assessment.

However, several security considerations were identified:

* Container images should use trusted base images.
* Base images should be regularly updated.
* Images should be scanned for vulnerabilities before production deployment.
* Containers should run with the minimum required privileges.
* Only required ports should be exposed.
* Sensitive credentials should not be stored inside Containerfiles or images.
* Image provenance should be verified in production environments.

## Overall Result

**Lab Status: PASS**

All required objectives were successfully demonstrated and supported by captured evidence.

The custom Nginx image was built, inspected, executed, tested, and verified successfully.
