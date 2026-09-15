# Lab 7 — Command Reference

## 1. Create Project

```bash
mkdir -p ~/07-Layer-Caching-and-Optimization
cd ~/07-Layer-Caching-and-Optimization
```

## 2. Create Application

```bash
cat > app.py <<'EOF'
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from optimized container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
EOF
```

## 3. Build Initial Image

```bash
podman build -t myapp:initial -f Dockerfile.initial .
```

## 4. Inspect Initial Image

```bash
podman images myapp:initial
```

```bash
podman history myapp:initial
```

```bash
podman inspect myapp:initial
```

```bash
podman image inspect myapp:initial --format '{{.Size}} bytes'
```

## 5. Build Optimized Image

```bash
podman build -t myapp:optimized -f Dockerfile.optimized .
```

## 6. Compare Images

```bash
podman images myapp:*
```

## 7. Inspect Optimized Image

```bash
podman history myapp:optimized
```

```bash
podman image inspect myapp:optimized --format '{{.Size}} bytes'
```

## 8. Test Layer Cache

```bash
cat >> app.py <<'EOF'

# Testing layer cache behavior
EOF
```

```bash
podman build -t myapp:optimized -f Dockerfile.optimized .
```

## 9. Cache Busting

```bash
podman build \
  -t myapp:optimized \
  --build-arg CACHEBUST=$(date +%s) \
  -f Dockerfile.optimized .
```

## 10. Multi-Stage Build

```bash
cat > requirements.txt <<'EOF'
Flask
EOF
```

```bash
podman build -t myapp:multistage -f Dockerfile.multistage .
```

## 11. Compare All Images

```bash
podman images myapp:*
```

```bash
podman history myapp:initial
```

```bash
podman history myapp:optimized
```

```bash
podman history myapp:multistage
```

## 12. Run Optimized Container

```bash
podman rm -f myapp-optimized 2>/dev/null || true
```

```bash
podman run -d \
  --name myapp-optimized \
  -p 8080:8080 \
  myapp:optimized
```

## 13. Verify Container

```bash
podman ps
```

```bash
podman port myapp-optimized
```

## 14. Test Application

```bash
curl http://localhost:8080
```

## 15. Check Logs

```bash
podman logs myapp-optimized
```

## 16. Verify Screenshots

```bash
ls -lh screenshots/
```

## Important Note

The command:

```bash
apt-get update && apt-get install ...
```

should not be executed directly on the host as a substitute for the Dockerfile `RUN` instruction. Package installation in this lab is performed during the image build.

The host-side execution produced a permission error because the command attempted to modify protected APT directories without root privileges. This did not affect the successfully built container images.
