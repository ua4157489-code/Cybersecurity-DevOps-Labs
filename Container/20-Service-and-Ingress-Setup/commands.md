# Lab 20 — Command Reference

## 1. Verify Kubernetes Environment

```bash
kubectl version --client
kind version
kubectl config current-context
kubectl get nodes -o wide
```

Expected active context:

```text
kind-lab20
```

---

## 2. Create Nginx Deployment

```bash
kubectl create deployment nginx --image=nginx:latest
```

Check rollout:

```bash
kubectl rollout status deployment/nginx
```

Verify:

```bash
kubectl get deployment nginx -o wide
kubectl get pods -l app=nginx -o wide
```

---

## 3. Verify Nginx Version

```bash
kubectl exec deployment/nginx -- nginx -v
```

Actual version:

```text
nginx version: nginx/1.31.6
```

---

## 4. Create ClusterIP Service

```bash
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP \
  --name=nginx-clusterip
```

Verify:

```bash
kubectl get service nginx-clusterip
kubectl describe service nginx-clusterip
kubectl get endpoints nginx-clusterip
```

Actual ClusterIP:

```text
10.96.210.14
```

Actual endpoint:

```text
10.244.0.5:80
```

---

## 5. Test ClusterIP

```bash
kubectl run curl-test \
  --image=curlimages/curl:8.10.1 \
  --restart=Never \
  --rm -it -- \
  curl -I http://nginx-clusterip
```

Test response body:

```bash
kubectl run curl-test \
  --image=curlimages/curl:8.10.1 \
  --restart=Never \
  --rm -it -- \
  curl -s http://nginx-clusterip | head
```

Actual result:

```text
HTTP/1.1 200 OK
```

---

## 6. Create NodePort Service

```bash
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=NodePort \
  --name=nginx-nodeport
```

Verify:

```bash
kubectl get service nginx-nodeport
kubectl get service nginx-nodeport -o jsonpath='{.spec.ports[0].nodePort}{"\n"}'
kubectl describe service nginx-nodeport
```

Actual NodePort:

```text
31787
```

---

## 7. Verify Kind Node Networking

```bash
kubectl get nodes -o wide
docker ps --filter "name=lab20"
docker inspect lab20-control-plane \
  --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

Actual Kind node IP:

```text
172.18.0.2
```

---

## 8. Test NodePort

```bash
docker exec lab20-control-plane \
  curl -I http://172.18.0.2:31787
```

Test response body:

```bash
docker exec lab20-control-plane \
  curl -s http://172.18.0.2:31787 | head
```

Actual result:

```text
HTTP/1.1 200 OK
```

---

## 9. Check Ingress Controller

```bash
kubectl get ingressclass
kubectl get pods -n ingress-nginx
```

Initially, no Ingress controller was installed.

---

## 10. Install Ingress-NGINX

```bash
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Wait for the controller:

```bash
kubectl wait \
  --namespace ingress-nginx \
  --for=condition=Ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=180s
```

Verify:

```bash
kubectl get pods -n ingress-nginx -o wide
kubectl get ingressclass
kubectl get service -n ingress-nginx
```

---

## 11. Create Ingress Manifest

```bash
cat > nginx-ingress.yaml <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-clusterip
            port:
              number: 80
EOF
```

---

## 12. Apply Ingress

```bash
kubectl apply -f nginx-ingress.yaml
```

Verify:

```bash
kubectl get ingress nginx-ingress
kubectl get ingress nginx-ingress -o wide
kubectl describe ingress nginx-ingress
```

Actual backend:

```text
nginx-clusterip:80
```

Actual host:

```text
nginx.example.com
```

---

## 13. Verify Ingress Controller Service

```bash
kubectl get service -n ingress-nginx ingress-nginx-controller
```

Actual ports:

```text
80:31089/TCP
443:31692/TCP
```

---

## 14. Test Ingress HTTP Routing

```bash
docker exec lab20-control-plane \
  curl -I \
  -H "Host: nginx.example.com" \
  http://127.0.0.1:31089
```

Test response body:

```bash
docker exec lab20-control-plane \
  curl -s \
  -H "Host: nginx.example.com" \
  http://127.0.0.1:31089 | head
```

Actual result:

```text
HTTP/1.1 200 OK
```

---

## 15. Test Through Kind Node IP

```bash
docker exec lab20-control-plane \
  curl -I \
  -H "Host: nginx.example.com" \
  http://172.18.0.2:31089
```

Actual result:

```text
HTTP/1.1 200 OK
```

---

## 16. Final Verification

```bash
kubectl get nodes -o wide
kubectl get deployment nginx -o wide
kubectl get pods -l app=nginx -o wide

kubectl get service nginx-clusterip
kubectl get endpoints nginx-clusterip

kubectl get service nginx-nodeport -o wide

kubectl get pods -n ingress-nginx -o wide
kubectl get ingressclass
kubectl get service -n ingress-nginx ingress-nginx-controller

kubectl get ingress nginx-ingress -o wide
kubectl describe ingress nginx-ingress

kubectl exec deployment/nginx -- nginx -v
```

---

## 17. Screenshot Directory

```bash
ls -lh screenshots/
```

Expected evidence files:

```text
screenshots/01-cluster-and-nginx-deployment.png
screenshots/02-clusterip-service-and-test.png
screenshots/03-nodeport-service-and-test.png
screenshots/04-ingress-controller-and-resource.png
screenshots/05-ingress-http-test.png
```
