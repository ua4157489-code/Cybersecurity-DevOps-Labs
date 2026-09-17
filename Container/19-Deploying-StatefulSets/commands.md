# Lab 19 — Command Reference

## 1. Create Lab Directory

```bash
mkdir -p ~/Alrazzaq_Labs/Container/19-Deploying-StatefulSets/screenshots
cd ~/Alrazzaq_Labs/Container/19-Deploying-StatefulSets
```

## 2. Verify Kubernetes Context

```bash
kubectl config current-context
```

Expected lab context:

```text
kind-lab19
```

## 3. Verify Cluster

```bash
kubectl get nodes
```

The lab used:

```text
lab19-control-plane
```

## 4. Check StorageClass

```bash
kubectl get storageclass
```

The default StorageClass was:

```text
standard
```

with the `rancher.io/local-path` provisioner.

## 5. Create StatefulSet Manifest

Create:

```bash
cat > mysql-statefulset.yaml <<'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
EOF
```

## 6. Validate Manifest

```bash
kubectl apply --dry-run=client -f mysql-statefulset.yaml
```

## 7. Create Headless Service Manifest

```bash
cat > mysql-service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  ports:
  - port: 3306
    name: mysql
  selector:
    app: mysql
EOF
```

## 8. Validate Service

```bash
kubectl apply --dry-run=client -f mysql-service.yaml
```

## 9. Deploy Service

```bash
kubectl apply -f mysql-service.yaml
```

The Kubernetes API reported a harmless warning that SessionAffinity is ignored for headless Services.

## 10. Deploy StatefulSet

```bash
kubectl apply -f mysql-statefulset.yaml
```

## 11. Verify StatefulSet

```bash
kubectl get statefulset
```

## 12. Verify Pods

```bash
kubectl get pods -l app=mysql -o wide
```

Final Pods:

```text
mysql-0
mysql-1
```

## 13. Verify PVCs

```bash
kubectl get pvc
```

Two 1Gi PVCs were created and became `Bound`.

## 14. Verify Service

```bash
kubectl get service mysql
```

The Service was headless:

```text
ClusterIP: None
```

## 15. Inspect StatefulSet Identity

```bash
kubectl get pods -l app=mysql \
  -o custom-columns="NAME:.metadata.name,HOSTNAME:.spec.hostname,SUBDOMAIN:.spec.subdomain,IP:.status.podIP"
```

## 16. Inspect EndpointSlice

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=mysql \
  -o wide
```

The EndpointSlice contained the StatefulSet Pod hostnames.

## 17. Verify MySQL Version

```bash
kubectl exec mysql-0 -- \
  mysql -uroot -ppassword -e "SELECT VERSION();"
```

Verified:

```text
8.0.46
```

## 18. Create Test Database

```bash
kubectl exec mysql-0 -- \
  mysql -uroot -ppassword -e "CREATE DATABASE lab_test;"
```

## 19. Verify Database

```bash
kubectl exec mysql-0 -- \
  mysql -uroot -ppassword -e "SHOW DATABASES;"
```

## 20. Delete mysql-0

```bash
kubectl delete pod mysql-0
```

The StatefulSet automatically recreated the Pod.

## 21. Verify Pod Recreation

```bash
kubectl get pods -l app=mysql -o wide
```

The recreated Pod retained the name:

```text
mysql-0
```

but received a new Pod IP.

## 22. Verify Persistent Data

```bash
kubectl exec mysql-0 -- \
  mysql -uroot -ppassword -e "SHOW DATABASES;"
```

The `lab_test` database remained present.

## 23. Verify StatefulSet DNS

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  --rm -it -- \
  nslookup mysql-0.mysql.default.svc.cluster.local
```

And:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  --rm -it -- \
  nslookup mysql-1.mysql.default.svc.cluster.local
```

Verified:

```text
mysql-0.mysql.default.svc.cluster.local -> 10.244.0.9
mysql-1.mysql.default.svc.cluster.local -> 10.244.0.8
```

## 24. Verify Final Lab State

```bash
kubectl get statefulset mysql
kubectl get pods -l app=mysql -o wide
kubectl get pvc
kubectl get service mysql
```

## 25. Cleanup StatefulSet

```bash
kubectl delete -f mysql-statefulset.yaml
```

## 26. Cleanup Service

```bash
kubectl delete -f mysql-service.yaml
```

## 27. Remove PVCs

If the persistent data is no longer required:

```bash
kubectl delete pvc \
  mysql-persistent-storage-mysql-0 \
  mysql-persistent-storage-mysql-1
```

## 28. Delete Kind Cluster

If the entire lab cluster is no longer required:

```bash
kind delete cluster --name lab19
```
