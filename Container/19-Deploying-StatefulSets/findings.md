# Lab 19 — Findings

## 1. StatefulSet Deployment

The `mysql` StatefulSet was successfully deployed with two replicas.

Final workload:

```text
mysql-0   Running
mysql-1   Running
```

The StatefulSet maintained the expected ordinal naming scheme.

## 2. Persistent Storage

Two PersistentVolumeClaims were automatically generated from the StatefulSet `volumeClaimTemplates`.

```text
mysql-persistent-storage-mysql-0
mysql-persistent-storage-mysql-1
```

Both PVCs were:

```text
Status: Bound
Capacity: 1Gi
Access Mode: RWO
StorageClass: standard
```

This demonstrated that each StatefulSet replica receives independent persistent storage.

## 3. MySQL Application

The deployed MySQL version was:

```text
8.0.46
```

The database server was operational and accessible through `kubectl exec`.

## 4. Data Persistence

A test database named:

```text
lab_test
```

was created on `mysql-0`.

The Pod was then deleted.

The StatefulSet recreated:

```text
mysql-0
```

The database was checked again after recreation and `lab_test` was still present.

### Finding

The test confirmed that the MySQL data was stored on persistent storage rather than being dependent on the lifecycle of the original Pod.

## 5. Stable Pod Identity

The StatefulSet maintained stable identities:

```text
mysql-0
mysql-1
```

After `mysql-0` was recreated, its name remained:

```text
mysql-0
```

Its IP address changed from the original address to:

```text
10.244.0.9
```

This demonstrates that StatefulSets provide stable identity without making Pod IP addresses permanent.

## 6. Headless Service

The MySQL Service was configured as:

```text
ClusterIP: None
```

This confirmed that it was a headless Service.

The Service selected the MySQL Pods using:

```text
app=mysql
```

## 7. Endpoint Discovery

The EndpointSlice associated with the Service contained the StatefulSet Pod identities.

The endpoints included:

```text
mysql-0
mysql-1
```

This provided evidence that the headless Service was correctly associated with the StatefulSet Pods.

## 8. DNS

The following FQDNs resolved successfully:

```text
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
```

Observed addresses:

```text
mysql-0.mysql.default.svc.cluster.local -> 10.244.0.9
mysql-1.mysql.default.svc.cluster.local -> 10.244.0.8
```

This verified DNS-based discovery of individual StatefulSet replicas.

## 9. Pod Recreation

The StatefulSet controller automatically recreated `mysql-0` after deletion.

The recreated Pod:

* Retained the same StatefulSet name.
* Retained its associated PVC.
* Received a new Pod IP.
* Retained access to the previously stored MySQL data.

## 10. Expected Warnings

The MySQL client displayed the standard warning associated with providing a password directly on the command line:

```text
Using a password on the command line interface can be insecure.
```

This did not affect the database test.

The Service creation also produced a warning indicating that SessionAffinity is ignored for headless Services. The Service was nevertheless created successfully.

## 11. Verification Notes

During the lab, Kubernetes resource commands were adjusted where necessary to match the installed `kubectl` behavior.

For example, separate resource queries were used rather than relying on a comma-separated resource command.

A direct `hostname` check inside the MySQL container was also not used as the identity verification method because the MySQL image did not contain the expected `hostname` executable.

Instead, StatefulSet identity was verified through:

* Kubernetes Pod metadata
* StatefulSet configuration
* EndpointSlice records
* StatefulSet FQDN resolution

## 12. Overall Findings

The lab confirmed the primary StatefulSet characteristics:

| Feature                     | Result   |
| --------------------------- | -------- |
| StatefulSet deployment      | Verified |
| Two replicas                | Verified |
| Stable Pod names            | Verified |
| Persistent PVCs             | Verified |
| Headless Service            | Verified |
| StatefulSet DNS             | Verified |
| Automatic Pod recreation    | Verified |
| Data persistence            | Verified |
| Independent replica storage | Verified |

## Conclusion

The StatefulSet successfully provided persistent storage and stable workload identity for the MySQL application.

The most significant verification was the deletion and recreation of `mysql-0`, after which the `lab_test` database remained available. This demonstrated practical data persistence through a Pod lifecycle event.
