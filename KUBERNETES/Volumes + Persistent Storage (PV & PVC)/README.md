# Kubernetes Volumes, PV & PVC Notes

# Table of Contents

1. What are Volumes?
2. Why Volumes are Needed
3. Types of Volumes
4. Volume vs Persistent Volume
5. Persistent Volume (PV)
6. Persistent Volume Claim (PVC)
7. Storage Class
8. Volume Binding
9. Using Volumes in Pods
10. EmptyDir Volume
11. HostPath Volume
12. ConfigMap & Secret Volumes
13. Dynamic Provisioning
14. Volume Lifecycle
15. Troubleshooting
16. Interview Questions
17. Scenario-Based Questions
18. Production Best Practices
19. Quick Revision

---

# 1. What are Volumes?

A **Volume** in Kubernetes is a directory accessible to containers in a Pod.

It solves:

* Data sharing between containers
* Data persistence (temporary or permanent)

---

# 2. Why Volumes are Needed

Containers are **ephemeral**:

* Data is lost when container restarts
* Pods can be rescheduled on different nodes

Volumes solve:

* Data persistence
* Data sharing between containers
* Configuration mounting

---

# 3. Types of Volumes

| Type                  | Purpose            |
| --------------------- | ------------------ |
| emptyDir              | Temporary storage  |
| hostPath              | Node-level storage |
| configMap             | Config injection   |
| secret                | Secure data        |
| persistentVolumeClaim | Persistent storage |

---

# 4. Volume vs Persistent Storage

| Volume                  | PV/PVC             |
| ----------------------- | ------------------ |
| Pod-level               | Cluster-level      |
| Temporary or node-based | Persistent storage |
| Tied to Pod lifecycle   | Independent of Pod |

---

# 5. Persistent Volume (PV)

A **PersistentVolume (PV)** is a storage resource in the cluster.

Characteristics:

* Independent of Pods
* Provisioned by admin or dynamically
* Reusable

---

# PV YAML

```yaml id="pv1"
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-demo

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /mnt/data
```

---

# 6. Persistent Volume Claim (PVC)

A **PVC** is a request for storage by a user.

Think:

* PV = actual storage
* PVC = request for storage

---

# PVC YAML

```yaml id="pvc1"
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: pvc-demo

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi
```

---

# 7. Storage Class

StorageClass defines:

* Type of storage
* Provisioner
* Dynamic provisioning rules

---

# Example

```yaml id="sc1"
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: kubernetes.io/aws-ebs
```

---

# 8. Volume Binding

PVC binds to PV automatically if:

* Size matches
* Access mode matches

States:

* Pending
* Bound
* Released

---

# 9. Using Volumes in Pods

```yaml id="podvol1"
apiVersion: v1
kind: Pod

metadata:
  name: volume-pod

spec:
  containers:
    - name: app
      image: nginx

      volumeMounts:
        - name: data-volume
          mountPath: /data

  volumes:
    - name: data-volume
      emptyDir: {}
```

---

# 10. emptyDir Volume

* Created when Pod starts
* Deleted when Pod stops

Use cases:

* Cache
* Temporary files

---

# 11. hostPath Volume

* Uses node filesystem
* Not recommended for production

Example:

```yaml id="host1"
hostPath:
  path: /data
```

Risks:

* Node dependency
* No portability

---

# 12. ConfigMap & Secret Volumes

## ConfigMap Volume

```yaml id="cmvol"
volumes:
  - name: config
    configMap:
      name: app-config
```

## Secret Volume

```yaml id="secvol"
volumes:
  - name: secret
    secret:
      secretName: db-secret
```

---

# 13. Dynamic Provisioning

When PVC is created:

* Kubernetes automatically creates PV
* Uses StorageClass

No manual admin intervention needed.

---

# 14. Volume Lifecycle

1. Provision
2. Bind PVC → PV
3. Mount into Pod
4. Use data
5. Release
6. Reclaim

---

# Reclaim Policies

| Policy  | Behavior       |
| ------- | -------------- |
| Retain  | Keep data      |
| Delete  | Remove storage |
| Recycle | Reuse volume   |

---

# 15. Troubleshooting

# Problem 1: PVC Pending

Check:

```bash id="t1"
kubectl get pvc
```

Possible reasons:

* No matching PV
* StorageClass missing
* Insufficient storage

---

# Problem 2: Pod Stuck in ContainerCreating

Check events:

```bash id="t2"
kubectl describe pod POD
```

---

# Problem 3: Volume Not Mounted

Check:

* volumeMount path
* volume name mismatch

---

# Problem 4: PV Not Binding

Check:

* Access modes
* Storage size mismatch

---

# 16. Interview Questions

## Beginner

### 1. What is a Volume?

Storage attached to a Pod.

---

### 2. Why are Volumes needed?

Containers are ephemeral.

---

### 3. What is emptyDir?

Temporary storage that exists as long as Pod exists.

---

### 4. What is PV?

Cluster-level persistent storage.

---

### 5. What is PVC?

Request for storage by a Pod.

---

## Intermediate

### 6. Difference between PV and PVC?

PV = actual storage
PVC = request for storage

---

### 7. What is StorageClass?

Defines dynamic storage provisioning.

---

### 8. What happens when PVC is deleted?

Depends on reclaim policy.

---

### 9. Can multiple Pods use same PVC?

Yes (depending on access mode).

---

### 10. What is hostPath?

Node-local storage (not recommended).

---

## Advanced

### 11. What is dynamic provisioning?

Automatic PV creation when PVC is requested.

---

### 12. What are access modes?

* ReadWriteOnce
* ReadOnlyMany
* ReadWriteMany

---

### 13. What happens if PV is deleted?

Depends on reclaim policy.

---

### 14. Why is storage abstraction important?

Decouples storage from application.

---

### 15. How does Kubernetes handle storage lifecycle?

Through PV controller and CSI drivers.

---

# 17. Scenario-Based Questions

## Scenario 1

App data is lost after Pod restart.

Answer:
Use PersistentVolume + PVC.

---

## Scenario 2

PVC stuck in Pending state.

Check:

* No matching PV
* StorageClass missing

---

## Scenario 3

Need shared storage between multiple Pods.

Answer:
Use RWX storage (NFS / cloud storage).

---

## Scenario 4

Logs need to persist.

Answer:
Use PVC mounted to log directory.

---

## Scenario 5

Temporary cache needed.

Answer:
Use emptyDir.

---

## Scenario 6

Database storage needed.

Answer:
Use StatefulSet + PVC.

---

## Scenario 7

Storage should auto-provision.

Answer:
Use StorageClass + dynamic provisioning.

---

# 18. Production Best Practices

## Do

* Use PVC instead of hostPath
* Use StorageClass
* Use cloud storage (EBS, Azure Disk)
* Use StatefulSets for databases
* Backup persistent volumes

---

## Avoid

* Using hostPath in production
* Storing data in container filesystem
* Using emptyDir for important data

---

# 19. Quick Revision

## Volume =

* Pod-level storage

## PV =

* Actual cluster storage

## PVC =

* Storage request

## StorageClass =

* Dynamic provisioning

---

# Most Important Interview Point

Pods are ephemeral, so:

👉 Always use PVC for real data persistence in production.
