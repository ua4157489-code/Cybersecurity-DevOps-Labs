# Lab 10: Commands

## 1. Create Named Volume

```bash
podman volume create myapp_data
podman volume ls
```

## 2. Inspect Volume

```bash
podman volume inspect myapp_data
```

## 3. Run Nginx with Named Volume

```bash
podman run -d --name webapp \
  -v myapp_data:/var/www/html \
  docker.io/library/nginx

podman exec webapp ls /var/www/html
```

## 4. Write Data to the Volume

```bash
podman exec webapp sh -c 'echo "Hello, Volume!" > /var/www/html/index.html'
podman exec webapp cat /var/www/html/index.html
```

Expected test data:

```text
Hello, Volume!
```

## 5. Recreate Container and Verify Persistence

```bash
podman rm -f webapp

podman run -d --name webapp_new \
  -v myapp_data:/var/www/html \
  docker.io/library/nginx

podman exec webapp_new cat /var/www/html/index.html
```

The previously stored data remained available:

```text
Hello, Volume!
```

## 6. Create Host Directory

```bash
mkdir -p ~/host_data
echo "Hello, Bind Mount!" > ~/host_data/index.html
cat ~/host_data/index.html
```

## 7. Run Nginx with Bind Mount

```bash
podman run -d --name bind_example \
  -v ~/host_data:/usr/share/nginx/html:Z \
  docker.io/library/nginx
```

## 8. Verify Bind-Mounted Data

```bash
podman exec bind_example cat /usr/share/nginx/html/index.html
```

## 9. Test Live Host Update

```bash
echo "Updated content!" >> ~/host_data/index.html
cat ~/host_data/index.html
podman exec bind_example cat /usr/share/nginx/html/index.html
```

## 10. Cleanup

Run these only after all screenshots and documentation have been completed:

```bash
podman rm -f webapp_new bind_example
podman volume rm myapp_data
rm -rf ~/host_data
```

## Troubleshooting

When initially writing `Hello, Volume!`, Bash history expansion caused:

```text
-bash: !': event not found
```

The command was corrected by using single quotes around the outer shell command:

```bash
podman exec webapp sh -c 'echo "Hello, Volume!" > /var/www/html/index.html'
```

This prevents Bash from interpreting `!` as history expansion.
