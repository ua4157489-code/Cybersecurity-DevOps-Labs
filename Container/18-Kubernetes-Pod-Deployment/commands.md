# Lab 18 — Command Reference

## 1. Verify Kubernetes Client

```bash
kubectl version --client
```

## 2. Verify Kubernetes Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl config current-context
```

## 3. Create Lab Directory

```bash
mkdir -p ~/Alrazzaq_Labs/Container/18-Kubernetes-Pod-Deployment
cd ~/Alrazzaq_Labs/Container/18-Kubernetes-Pod-Deployment
```

## 4. Create Pod Manifest

```bash
cat > simple-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
EOF
```

## 5. Verify Manifest

```bash
cat simple-pod.yaml
```

## 6. Validate Manifest

```bash
kubectl apply --dry-run=client -f simple-pod.yaml
```

## 7. Deploy Pod

```bash
kubectl apply -f simple-pod.yaml
```

## 8. Check Pod Status

```bash
kubectl get pods -o wide
```

## 9. Inspect Pod YAML

```bash
kubectl get pod nginx-pod -o yaml
```

## 10. Inspect Pod Details

```bash
kubectl describe pod nginx-pod
```

## 11. View Kubernetes Events

```bash
kubectl get events --sort-by='.lastTimestamp'
```

## 12. View Pod Logs

```bash
kubectl logs nginx-pod
```

## 13. Test Nginx Configuration

```bash
kubectl exec nginx-pod -- nginx -t
```

## 14. Test Nginx Internally

```bash
kubectl exec nginx-pod -- curl -I http://localhost
```

## 15. Port Forward

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

## 16. Test Port Forward

From another terminal:

```bash
curl -I http://127.0.0.1:8080
```

```bash
curl http://127.0.0.1:8080
```

## 17. Final Verification

```bash
kubectl get nodes -o wide
kubectl get pod nginx-pod -o wide
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

## 18. Stop Port Forward

If running in the foreground:

```text
Ctrl+C
```

If running as a background process:

```bash
pkill -f "kubectl port-forward pod/nginx-pod"
```
