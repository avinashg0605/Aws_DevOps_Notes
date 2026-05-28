# Kubernetes StatefulSets Notes

# Table of Contents

1. What is StatefulSet?
2. Why StatefulSets are needed
3. StatefulSet vs Deployment
4. StatefulSet Architecture
5. StatefulSet Features
6. Headless Service (Important)
7. StatefulSet YAML
8. Pod Identity (Stable naming)
9. Storage with StatefulSets
10. Scaling StatefulSets
11. Rolling Updates
12. Deleting StatefulSets
13. Use Cases
14. Troubleshooting
15. Interview Questions
16. Scenario-Based Questions
17. Production Best Practices
18. Quick Revision

---

# 1. What is StatefulSet?

A **StatefulSet** is a Kubernetes workload controller used to manage **stateful applications** that require:

* Stable network identity
* Persistent storage
* Ordered deployment and scaling

It is part of Kubernetes.

---

# 2. Why StatefulSets are needed

Stateless apps (like web servers):

* Can be replaced easily
* No identity required

Stateful apps (like databases):

* Need stable identity
* Need persistent storage
* Must maintain order

Examples:

* MySQL
* MongoDB
* Kafka
* Elasticsearch

---

# 3. StatefulSet vs Deployment

| Feature      | Deployment       | StatefulSet           |
| ------------ | ---------------- | --------------------- |
| Pod Identity | Random           | Stable (pod-0, pod-1) |
| Storage      | Shared/ephemeral | Persistent per Pod    |
| Order        | Parallel         | Ordered               |
| Use case     | Stateless apps   | Stateful apps         |

---

# 4. StatefulSet Architecture

```text
StatefulSet
    ↓
Headless Service
    ↓
Pods (ordered)
    ↓
PVC per Pod
```

---

# 5. StatefulSet Features

* Stable Pod names
* Stable DNS names
* Ordered deployment
* Ordered termination
* Persistent storage per Pod

---

# 6. Headless Service (VERY IMPORTANT)

StatefulSets require a **Headless Service**:

```yaml
clusterIP: None
```

Why?

* Gives direct Pod DNS resolution
* No load balancing
* Each Pod has unique identity

---

# 7. StatefulSet YAML

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql

spec:
  serviceName: mysql-service
  replicas: 3

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
          image: mysql:5.7
          ports:
            - containerPort: 3306

          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: data

      spec:
        accessModes: ["ReadWriteOnce"]

        resources:
          requests:
            storage: 1Gi
```

---

# 8. Pod Identity (Very Important)

StatefulSet Pods get predictable names:

```text
mysql-0
mysql-1
mysql-2
```

Each Pod:

* Has unique identity
* Maintains same name after restart

---

# DNS Naming Example

```text
mysql-0.mysql-service.default.svc.cluster.local
mysql-1.mysql-service.default.svc.cluster.local
```

---

# 9. Storage with StatefulSets

Each Pod gets its own PVC:

| Pod     | PVC          |
| ------- | ------------ |
| mysql-0 | data-mysql-0 |
| mysql-1 | data-mysql-1 |
| mysql-2 | data-mysql-2 |

This ensures:

* No data loss
* Independent storage per replica

---

# 10. Scaling StatefulSets

## Scale Up

```bash
kubectl scale statefulset mysql --replicas=5
```

Pods created in order:

* mysql-3
* mysql-4

---

## Scale Down

Pods deleted in reverse order:

* mysql-4 first
* then mysql-3

---

# 11. Rolling Updates

StatefulSet updates are:

* Ordered
* One Pod at a time

Ensures:

* Data consistency
* Zero corruption risk

---

# 12. Deleting StatefulSets

## Delete StatefulSet only

```bash
kubectl delete statefulset mysql
```

## Delete with PVCs

PVCs remain unless manually deleted.

---

# 13. Use Cases

Best suited for:

* Databases (MySQL, PostgreSQL)
* Distributed systems (Kafka, Zookeeper)
* Stateful apps
* Queues

---

# 14. Troubleshooting

## Problem 1: Pod stuck in Pending

Check PVC:

```bash
kubectl get pvc
```

Cause:

* Storage not available

---

## Problem 2: Pod not starting in order

Check:

* Headless service
* StatefulSet config

---

## Problem 3: DNS not resolving

Check:

* Headless service exists
* Correct serviceName

---

## Problem 4: Data lost after restart

Cause:

* Missing PVC
* Wrong volumeClaimTemplates

---

# 15. Interview Questions

## Beginner Level

### 1. What is StatefulSet?

Workload for stateful applications with stable identity.

---

### 2. Difference between Deployment and StatefulSet?

Deployment:

* Stateless
* Random Pods

StatefulSet:

* Stateful
* Stable Pod names

---

### 3. What is Pod identity in StatefulSet?

Predictable names like mysql-0, mysql-1.

---

### 4. Why do we need StatefulSet?

To manage databases and stateful apps.

---

### 5. What is Headless Service?

Service without ClusterIP for direct Pod access.

---

## Intermediate Level

### 6. Why ordered deployment?

To maintain consistency in distributed systems.

---

### 7. What happens when a StatefulSet Pod restarts?

Same name and same PVC.

---

### 8. Can StatefulSet scale in parallel?

No, it scales sequentially.

---

### 9. What is volumeClaimTemplates?

Template for automatic PVC creation per Pod.

---

### 10. Does StatefulSet support rolling updates?

Yes, but sequentially.

---

## Advanced Level

### 11. Why is DNS important in StatefulSets?

Provides stable network identity.

---

### 12. What happens if headless service is deleted?

StatefulSet DNS breaks.

---

### 13. Why not use Deployment for databases?

No stable identity or storage guarantees.

---

### 14. How does Kubernetes ensure order?

Using StatefulSet controller logic.

---

### 15. How are PVCs handled in scaling?

Each Pod gets its own PVC.

---

# 16. Scenario-Based Questions

## Scenario 1

Database pod restarted but data is missing.

Answer:
PVC not attached properly or missing.

---

## Scenario 2

Need 3-node MongoDB cluster.

Answer:
Use StatefulSet with headless service.

---

## Scenario 3

Pods must have fixed names.

Answer:
StatefulSet provides stable identity.

---

## Scenario 4

Need safe rolling updates for database.

Answer:
Use StatefulSet rolling update strategy.

---

## Scenario 5

Need shared storage across all Pods.

Answer:
Not default StatefulSet behavior; use RWX storage.

---

## Scenario 6

Pod-1 must start before Pod-2.

Answer:
StatefulSet ensures ordered deployment.

---

# 17. Production Best Practices

## Do

* Always use StatefulSet for databases
* Use PVC per Pod
* Use Headless Services
* Backup persistent storage
* Monitor storage usage

---

## Avoid

* Using Deployment for databases
* Sharing same PVC across replicas (unless RWX)
* Deleting PVC accidentally

---

# 18. Quick Revision

## StatefulSet =

* For stateful apps
* Stable identity
* Persistent storage
* Ordered deployment

---

# Most Important Interview Point

👉 StatefulSet = Deployment + Stable Identity + Persistent Storage + Ordered Scaling
