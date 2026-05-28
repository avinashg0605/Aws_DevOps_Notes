# Kubernetes DaemonSets Notes

# Table of Contents

1. What is a DaemonSet?
2. Why DaemonSets are needed
3. How DaemonSet works
4. DaemonSet vs Deployment
5. DaemonSet Architecture
6. DaemonSet YAML
7. Scheduling behavior
8. Node selectors and tolerations
9. Updating DaemonSets
10. Deleting DaemonSets
11. Use cases
12. Troubleshooting
13. Interview Questions
14. Scenario-Based Questions
15. Production Best Practices
16. Quick Revision

---

# 1. What is a DaemonSet?

A **DaemonSet** ensures that a copy of a Pod runs on **every node** (or selected nodes) in a Kubernetes cluster.

It is part of Kubernetes.

---

# 2. Why DaemonSets are needed

In many real-world systems, you need one Pod per node for tasks like:

* Log collection
* Monitoring
* Node health checks
* Networking agents

Instead of manually deploying Pods on every node, DaemonSet automates it.

---

# 3. How DaemonSet works

Whenever a new node is added:

* DaemonSet automatically schedules a Pod on it

When a node is removed:

* Corresponding Pod is deleted

---

# 4. DaemonSet vs Deployment

| Feature          | Deployment     | DaemonSet           |
| ---------------- | -------------- | ------------------- |
| Pod distribution | Random         | One per node        |
| Scaling          | Manual/auto    | Node-based          |
| Use case         | Apps           | System-level agents |
| Replicas         | Defined number | Equal to nodes      |

---

# 5. DaemonSet Architecture

```text
Node 1 → Pod
Node 2 → Pod
Node 3 → Pod
```

Each node runs exactly one Pod (by default).

---

# 6. DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: fluentd-daemonset

spec:
  selector:
    matchLabels:
      app: fluentd

  template:
    metadata:
      labels:
        app: fluentd

    spec:
      containers:
        - name: fluentd
          image: fluent/fluentd:v1.16
```

---

# 7. Scheduling behavior

DaemonSets:

* Automatically bypass normal scheduling rules
* Ensure Pod runs on all nodes
* Use kube-scheduler internally with constraints

---

# 8. Node Selectors and Tolerations

You can restrict DaemonSet to specific nodes.

## Node Selector Example

```yaml
nodeSelector:
  disktype: ssd
```

---

## Tolerations Example

Used to run on tainted nodes (like master nodes):

```yaml
tolerations:
  - key: "node-role.kubernetes.io/master"
    effect: NoSchedule
```

---

# 9. Updating DaemonSets

DaemonSets support rolling updates.

Update image:

```bash
kubectl set image daemonset/fluentd fluentd=fluent/fluentd:v2
```

Update strategy:

* RollingUpdate (default)
* OnDelete

---

# 10. Deleting DaemonSets

```bash
kubectl delete daemonset fluentd-daemonset
```

This removes Pods from all nodes.

---

# 11. Use Cases

DaemonSets are used for cluster-wide services:

### Monitoring

* Prometheus node exporter

### Logging

* Fluentd
* Logstash

### Networking

* Calico
* Weave Net

### Security

* Security agents
* Antivirus scanners

---

# 12. Troubleshooting

## Problem 1: Pod not running on some nodes

Check:

* Node labels
* Taints and tolerations

---

## Problem 2: DaemonSet not scheduling on master node

Cause:

* Master node is tainted

Fix:

* Add toleration

---

## Problem 3: Pod not created on new node

Check:

* DaemonSet controller status
* Node readiness

---

## Problem 4: Image pull errors

Check:

```bash
kubectl describe pod POD_NAME
```

---

# 13. Interview Questions

## Beginner Level

### 1. What is DaemonSet?

Ensures one Pod per node.

---

### 2. When do we use DaemonSet?

For node-level services like logging and monitoring.

---

### 3. Does DaemonSet create multiple replicas per node?

No, one Pod per node by default.

---

### 4. What happens when a new node is added?

DaemonSet automatically creates Pod on it.

---

### 5. Can DaemonSet run on specific nodes?

Yes, using nodeSelector or tolerations.

---

## Intermediate Level

### 6. Difference between DaemonSet and Deployment?

Deployment:

* App-level workload
* Random distribution

DaemonSet:

* Node-level workload
* One Pod per node

---

### 7. How does DaemonSet handle scheduling?

Automatically assigns Pods to all eligible nodes.

---

### 8. Can DaemonSet be scaled manually?

No, it scales automatically based on nodes.

---

### 9. What is use of tolerations in DaemonSet?

To allow Pods to run on tainted nodes.

---

### 10. What happens when node is removed?

DaemonSet Pod on that node is deleted.

---

## Advanced Level

### 11. How does DaemonSet differ internally from ReplicaSet?

ReplicaSet maintains replicas; DaemonSet maintains node coverage.

---

### 12. Can DaemonSet run multiple Pods per node?

Yes, if configured manually with advanced scheduling, but default is one.

---

### 13. What is update strategy in DaemonSet?

RollingUpdate or OnDelete.

---

### 14. Can DaemonSet be used for stateless apps?

Not recommended.

---

### 15. What happens during cluster upgrade?

DaemonSet ensures agents run on new nodes automatically.

---

# 14. Scenario-Based Questions

## Scenario 1

Need logging agent on every node.

Answer:
Use DaemonSet with Fluentd.

---

## Scenario 2

Monitoring tool missing on new node.

Answer:
DaemonSet automatically deploys it.

---

## Scenario 3

Pod not running on master node.

Answer:
Add toleration for master taint.

---

## Scenario 4

Need antivirus agent on all nodes.

Answer:
Use DaemonSet.

---

## Scenario 5

Need different config on GPU nodes.

Answer:
Use nodeSelector.

---

## Scenario 6

One node has too many DaemonSet pods.

Answer:
Check duplicate DaemonSets or overlapping selectors.

---

# 15. Production Best Practices

## Do

* Use DaemonSets for system-level services
* Use nodeSelector for targeted deployment
* Add tolerations for master nodes if needed
* Monitor DaemonSet health

---

## Avoid

* Using DaemonSet for application workloads
* Running multiple conflicting DaemonSets on same nodes
* Ignoring resource limits

---

# 16. Quick Revision

## DaemonSet =

* One Pod per node
* Automatically scales with nodes
* Used for system-level services

---

# Most Important Interview Point

👉 DaemonSet ensures **one Pod per node automatically**, unlike Deployments which manage replicas.
