# Kubernetes ReplicaSets Notes

# Table of Contents

1. What is ReplicaSet?
2. Why ReplicaSets are Needed
3. How ReplicaSet Works
4. ReplicaSet Architecture
5. ReplicaSet YAML
6. Important Fields
7. Labels and Selectors
8. ReplicaSet Lifecycle
9. Creating ReplicaSets
10. Scaling ReplicaSets
11. Updating ReplicaSets
12. Deleting ReplicaSets
13. ReplicaSet vs ReplicationController
14. ReplicaSet vs Deployment
15. Self-Healing
16. Common kubectl Commands
17. Best Practices
18. Troubleshooting
19. Interview Questions
20. Scenario-Based Questions
21. Production Tips
22. Quick Revision

---

# 1. What is ReplicaSet?

A **ReplicaSet** is a Kubernetes controller that ensures a specified number of Pod replicas are always running.

Main purpose:

* Maintain desired Pod count
* Automatically recreate failed Pods
* Ensure application availability

---

# 2. Why ReplicaSets are Needed

Pods are ephemeral:

* Pods can crash
* Nodes can fail
* Containers may terminate unexpectedly

Without ReplicaSets:

* Manual Pod recreation required

ReplicaSet solves this by continuously monitoring Pods.

Example:

* Desired replicas = 3
* One Pod crashes
* ReplicaSet automatically creates a new Pod

---

# 3. How ReplicaSet Works

ReplicaSet continuously compares:

```text id="dyj0u8"
Desired State vs Actual State
```

If:

* Actual Pods < Desired Pods → Create Pods
* Actual Pods > Desired Pods → Delete Pods

---

# 4. ReplicaSet Architecture

```text id="m1jkf4"
ReplicaSet
    ↓
Pod Template
    ↓
Pods
```

ReplicaSet uses:

* Labels
* Selectors

to identify Pods it manages.

---

# 5. ReplicaSet YAML

```yaml id="yv4f1p"
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest

          ports:
            - containerPort: 80
```

---

# 6. Important Fields

| Field      | Purpose                  |
| ---------- | ------------------------ |
| replicas   | Desired number of Pods   |
| selector   | Identifies matching Pods |
| template   | Pod template             |
| labels     | Pod identification       |
| containers | Container configuration  |

---

# 7. Labels and Selectors

ReplicaSet manages Pods using labels.

## Pod Labels

```yaml id="q9v7tc"
labels:
  app: nginx
```

## Selector

```yaml id="tw5e2n"
selector:
  matchLabels:
    app: nginx
```

ReplicaSet controls Pods matching the selector.

---

# 8. ReplicaSet Lifecycle

ReplicaSet operations:

1. Create ReplicaSet
2. ReplicaSet creates Pods
3. Monitor Pod health
4. Replace failed Pods
5. Scale Pods up/down

---

# 9. Creating ReplicaSets

## Apply YAML

```bash id="sm8v7f"
kubectl apply -f replicaset.yaml
```

---

## Verify ReplicaSet

```bash id="k7p4d1"
kubectl get rs
```

OR

```bash id="tx0m2q"
kubectl get replicasets
```

---

## View Pods

```bash id="n8x5tw"
kubectl get pods
```

---

# 10. Scaling ReplicaSets

## Scale Up

```bash id="x3c8l2"
kubectl scale rs nginx-rs --replicas=5
```

---

## Scale Down

```bash id="j1v7e4"
kubectl scale rs nginx-rs --replicas=2
```

ReplicaSet automatically:

* Creates Pods
* Deletes Pods

to match desired state.

---

# 11. Updating ReplicaSets

ReplicaSets are NOT ideal for rolling updates.

Update image:

```bash id="r8v3pq"
kubectl edit rs nginx-rs
```

However:

* Existing Pods may not update correctly
* Deployments are preferred

---

# 12. Deleting ReplicaSets

## Delete ReplicaSet and Pods

```bash id="d5w2nq"
kubectl delete rs nginx-rs
```

---

## Delete ReplicaSet Only

```bash id="g9u1xb"
kubectl delete rs nginx-rs --cascade=orphan
```

Pods remain running.

---

# 13. ReplicaSet vs ReplicationController

| Feature                  | ReplicaSet | ReplicationController |
| ------------------------ | ---------- | --------------------- |
| Label Selectors          | Advanced   | Basic                 |
| Equality-based Selectors | Yes        | Yes                   |
| Set-based Selectors      | Yes        | No                    |
| Recommended              | Yes        | Deprecated            |

---

# 14. ReplicaSet vs Deployment

| Feature         | ReplicaSet      | Deployment   |
| --------------- | --------------- | ------------ |
| Pod Management  | Yes             | Yes          |
| Scaling         | Yes             | Yes          |
| Rolling Updates | Limited         | Full support |
| Rollbacks       | No              | Yes          |
| Production Use  | Rarely directly | Recommended  |

---

# 15. Self-Healing

ReplicaSet continuously monitors Pods.

Example:

* Pod deleted manually
* Node failure occurs
* Container crashes

ReplicaSet automatically recreates missing Pods.

---

# Example

Delete Pod:

```bash id="k3p8c4"
kubectl delete pod POD_NAME
```

ReplicaSet immediately creates replacement Pod.

---

# 16. Common kubectl Commands

| Command                            | Description        |
| ---------------------------------- | ------------------ |
| kubectl get rs                     | List ReplicaSets   |
| kubectl describe rs NAME           | ReplicaSet details |
| kubectl scale rs NAME --replicas=5 | Scale ReplicaSet   |
| kubectl delete rs NAME             | Delete ReplicaSet  |
| kubectl get pods                   | View Pods          |

---

# 17. Best Practices

## Do

* Use Deployments instead of standalone ReplicaSets
* Use labels properly
* Define resource limits
* Monitor replica counts

---

## Avoid

* Managing ReplicaSets manually in production
* Using overlapping selectors
* Using latest image tags

---

# 18. Troubleshooting

# Problem 1: Pods Not Creating

## Check ReplicaSet

```bash id="s8n1fw"
kubectl describe rs nginx-rs
```

Possible causes:

* Image pull failure
* Invalid YAML
* Resource shortage

---

# Problem 2: Pods Stuck Pending

Check:

```bash id="m7x2q9"
kubectl describe pod POD_NAME
```

Possible reasons:

* Insufficient CPU/memory
* PVC issues
* Node taints

---

# Problem 3: Replica Count Incorrect

Verify selector labels.

ReplicaSet may accidentally manage wrong Pods.

---

# Problem 4: Pods Continuously Restarting

Check logs:

```bash id="p5c2z8"
kubectl logs POD_NAME
```

Possible causes:

* Application crash
* Bad configuration
* Failed probes

---

# 19. Interview Questions

## Beginner Level

### 1. What is ReplicaSet?

ReplicaSet ensures desired number of Pod replicas are running.

---

### 2. Why do we need ReplicaSets?

To provide self-healing and maintain availability.

---

### 3. How does ReplicaSet identify Pods?

Using labels and selectors.

---

### 4. What happens if a Pod crashes?

ReplicaSet automatically creates replacement Pod.

---

### 5. Can ReplicaSet scale Pods?

Yes.

---

# Intermediate Level

### 6. Difference between ReplicaSet and Deployment?

ReplicaSet:

* Basic Pod replication

Deployment:

* ReplicaSets + rolling updates + rollback

---

### 7. Can ReplicaSet perform rolling updates?

Limited support. Deployments are preferred.

---

### 8. What is selector.matchLabels?

Used to identify Pods managed by ReplicaSet.

---

### 9. What happens if replicas are reduced?

Extra Pods are terminated.

---

### 10. Can two ReplicaSets manage same Pods?

Potentially yes if selectors overlap, which causes issues.

---

# Advanced Level

### 11. What is self-healing in Kubernetes?

Automatic recreation of failed Pods.

---

### 12. What happens during node failure?

ReplicaSet recreates Pods on healthy nodes.

---

### 13. Why are Deployments preferred over ReplicaSets?

Deployments provide rollout and rollback capabilities.

---

### 14. Explain desired state management.

Kubernetes continuously ensures actual state matches desired state.

---

### 15. How does ReplicaSet interact with scheduler?

ReplicaSet creates Pods; scheduler assigns them to nodes.

---

# 20. Scenario-Based Questions

# Scenario 1

## Question

One Pod suddenly disappears. What happens?

## Answer

ReplicaSet detects missing Pod and automatically creates new Pod.

---

# Scenario 2

## Question

Application traffic increased heavily. What should you do?

## Answer

Scale ReplicaSet:

```bash id="j7d4v1"
kubectl scale rs nginx-rs --replicas=10
```

---

# Scenario 3

## Question

Pods are being created continuously in loop.

## Answer

Possible causes:

* CrashLoopBackOff
* Bad application image
* Failed liveness probe

Check logs and events.

---

# Scenario 4

## Question

ReplicaSet is not managing Pods.

## Answer

Check:

* Labels
* Selectors
* Namespace mismatch

---

# Scenario 5

## Question

A node failed. How is application availability maintained?

## Answer

ReplicaSet recreates Pods on healthy nodes.

---

# Scenario 6

## Question

Developer accidentally deletes Pods manually.

## Answer

ReplicaSet automatically recreates deleted Pods.

---

# Scenario 7

## Question

Need zero-downtime upgrades.

## Answer

Use Deployment instead of standalone ReplicaSet.

---

# 21. Production Tips

## Recommended Approach

Production setup:

```text id="n8s3kw"
Deployment → ReplicaSet → Pods
```

Do NOT directly manage ReplicaSets in production unless required.

---

## Use Resource Limits

Always define:

* CPU requests
* Memory requests
* Limits

---

## Use Health Probes

Add:

* Liveness probe
* Readiness probe

---

## Avoid Overlapping Selectors

Multiple ReplicaSets controlling same Pods causes instability.

---

## Monitor Replica Health

Use:

* Prometheus
* Grafana

for monitoring.

---

# 22. Quick Revision

## ReplicaSet =

* Maintains desired Pod count
* Provides self-healing
* Uses labels/selectors
* Supports scaling

---

# Key Concepts

* Desired state
* Labels
* Selectors
* Self-healing
* Scaling

---

# Important Commands

```bash id="r0x5pm"
kubectl get rs
kubectl describe rs NAME
kubectl scale rs NAME --replicas=5
kubectl delete rs NAME
```

---

# Most Important Interview Point

Deployments internally manage ReplicaSets.

In real production:

* Use Deployments
* Rarely use ReplicaSets directly
