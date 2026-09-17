# Lab 19 — Deploying StatefulSets

## 1. Objective

The objective of this lab was to understand how Kubernetes StatefulSets manage stateful applications.

The practical objectives were:

* Deploy a MySQL StatefulSet.
* Create persistent storage for each replica.
* Provide stable Pod identities.
* Configure a headless Service.
* Verify StatefulSet DNS.
* Test data persistence after Pod deletion.
* Observe StatefulSet Pod recreation behavior.

## 2. Lab Environment

The lab was performed on a local Kind Kubernetes cluster.

```text
Kind Cluster: lab19
Context: kind-lab19
Kubernetes: v1.34.0
kubectl: v1.36.2
Kind: v0.30.0
Docker: 29.6.0
Node: lab19-control-plane
```

The cluster used the default `standard` StorageClass backed by the Kind local-path provisioner.

## 3. StatefulSet Design

The workload was defined as a MySQL StatefulSet with two replicas.

```text
StatefulSet
    │
    ├── mysql-0
    │     └── mysql-persistent-storage-mysql-0
    │
    └── mysql-1
          └── mysql-persistent-storage-mysql-1
```

Each replica received its own PersistentVolumeClaim through the `volumeClaimTemplates` configuration.

The requested storage size was 1Gi per replica.

## 4. Headless Service

A headless Service named `mysql` was created with:

```yaml
clusterIP: None
```

Unlike a standard ClusterIP Service, the headless Service does not provide a single virtual IP.

Instead, it allows DNS records to identify individual StatefulSet Pods.

## 5. Deployment Procedure

The Service was created first, followed by the StatefulSet.

The StatefulSet controller then created the Pods:

```text
mysql-0
mysql-1
```

PersistentVolumeClaims were automatically generated from the StatefulSet volume template.

The initial storage provisioning followed the cluster's `WaitForFirstConsumer` behavior. Once the Pods were scheduled, both PVCs became bound and the MySQL Pods entered the `Running` state.

## 6. Application Verification

After deployment, the MySQL server was accessed through `kubectl exec`.

The installed MySQL version was verified as:

```text
8.0.46
```

A database named:

```text
lab_test
```

was created to provide a persistent-data test.

## 7. Persistence Test

The `mysql-0` Pod was deleted:

```bash
kubectl delete pod mysql-0
```

Because the Pod belonged to a StatefulSet, Kubernetes automatically recreated it.

The recreated Pod retained the StatefulSet identity:

```text
mysql-0
```

However, its Pod IP changed.

The PersistentVolumeClaim associated with `mysql-0` remained available.

The MySQL database list was then checked again.

The previously created:

```text
lab_test
```

database was still present.

This confirmed that the database data persisted independently of the original Pod instance.

## 8. Stable Identity Test

Pod metadata was inspected using custom columns:

```text
NAME
HOSTNAME
SUBDOMAIN
IP
```

The StatefulSet provided stable identities:

```text
mysql-0
mysql-1
```

The Pod IPs were treated as temporary network addresses rather than stable identifiers.

This is one of the key differences between StatefulSets and ordinary Kubernetes Pods.

## 9. DNS Verification

The headless Service was used to verify StatefulSet DNS.

The following fully qualified DNS names were tested:

```text
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
```

Both names resolved successfully.

The final observed addresses were:

```text
mysql-0.mysql.default.svc.cluster.local -> 10.244.0.9
mysql-1.mysql.default.svc.cluster.local -> 10.244.0.8
```

The use of FQDNs provided reliable verification of the StatefulSet DNS records.

## 10. Verification Strategy

The lab verification was divided into four areas:

### Deployment

Verified:

* StatefulSet exists.
* Two replicas are running.
* MySQL containers are healthy.

### Storage

Verified:

* Two PVCs were created.
* Both PVCs were `Bound`.
* Each PVC requested 1Gi.

### Identity

Verified:

* `mysql-0` and `mysql-1` identities.
* StatefulSet hostname and subdomain.
* Pod identity remained after recreation.

### Persistence

Verified:

* `lab_test` database was created.
* `mysql-0` was deleted.
* StatefulSet recreated the Pod.
* `lab_test` remained available.

## 11. DNS Observation

A short-form lookup using BusyBox produced unsuccessful results for the individual names:

```text
mysql-0.mysql
mysql-1.mysql
```

The fully qualified Kubernetes DNS names were then tested and resolved successfully.

Therefore, the final DNS verification used:

```text
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
```

## 12. Result

All primary lab objectives were successfully demonstrated.

The StatefulSet provided:

* Stable Pod names
* Persistent storage
* Automatic Pod recreation
* Stable DNS identities
* Independent storage for each replica

The persistence test demonstrated that application data survives the lifecycle of an individual Pod.
