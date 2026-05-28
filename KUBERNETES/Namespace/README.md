# Kubernetes Namespace Notes

## What is a Namespace?

A **Namespace** in Kubernetes is a logical partition inside a cluster used to organize and isolate resources.

Namespaces help:

* Separate environments
* Organize workloads
* Control access
* Apply resource limits

---

# Why Namespaces are Needed

In large Kubernetes clusters:

* Multiple teams share the same cluster
* Applications may have same resource names
* Isolation is required

Namespaces solve these problems.

Example:

* `dev`
* `testing`
* `production`

---

# Default Namespaces in Kubernetes

| Namespace       | Purpose                       |
| --------------- | ----------------------------- |
| default         | Default namespace for objects |
| kube-system     | Kubernetes system components  |
| kube-public     | Publicly accessible resources |
| kube-node-lease | Node heartbeat information    |

---

# Creating a Namespace

## Command

```bash id="h41qez"
kubectl create namespace dev
```

---

# Namespace YAML Example

```yaml id="c84v0z"
apiVersion: v1
kind: Namespace

metadata:
  name: production
```

Apply:

```bash id="gh2r0d"
kubectl apply -f namespace.yaml
```

---

# Viewing Namespaces

## List Namespaces

```bash id="g1hzxk"
kubectl get namespaces
```

OR

```bash id="7xbo0p"
kubectl get ns
```

---

# Using a Namespace

## Create Resource in Namespace

```bash id="2j97n0"
kubectl apply -f pod.yaml -n dev
```

---

## View Resources in Namespace

```bash id="vx5r7s"
kubectl get pods -n dev
```

---

# Setting Default Namespace

Using context:

```bash id="2sy5e9"
kubectl config set-context --current --namespace=dev
```

Now all commands use `dev` namespace by default.

---

# Namespace Isolation

Resources inside one namespace are isolated from others.

Example:

* Pod in `dev`
* Pod in `prod`

can have same name.

---

# Resource Quotas

Namespaces can limit:

* CPU
* Memory
* Number of Pods

Example:

```yaml id="s0pxq2"
apiVersion: v1
kind: ResourceQuota

metadata:
  name: compute-quota

spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
```

---

# Limit Ranges

LimitRanges enforce:

* Default resource requests
* Maximum limits

Example:

```yaml id="k7y1n4"
apiVersion: v1
kind: LimitRange

metadata:
  name: resource-limits

spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"

      defaultRequest:
        cpu: "250m"
        memory: "256Mi"

      type: Container
```

---

# Namespace DNS

Service DNS format:

```text id="3mzv0h"
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text id="2k9m3n"
nginx-service.dev.svc.cluster.local
```

---

# Deleting Namespace

```bash id="t0knvh"
kubectl delete namespace dev
```

Deleting namespace removes all resources inside it.

---

# Best Practices

## Do

* Use namespaces for environments
* Apply resource quotas
* Use RBAC with namespaces
* Organize applications logically

## Avoid

* Putting everything in default namespace
* Using too many unnecessary namespaces

---

# Common kubectl Commands

| Command                  | Description         |
| ------------------------ | ------------------- |
| kubectl get ns           | List namespaces     |
| kubectl create ns NAME   | Create namespace    |
| kubectl delete ns NAME   | Delete namespace    |
| kubectl get pods -n NAME | View namespace Pods |

---

# Quick Revision

## Namespace =

* Logical cluster partition
* Resource isolation
* Multi-team organization
* Access control boundary

---

# Important Concepts

* Isolation
* ResourceQuota
* LimitRange
* RBAC
* DNS naming

---

# Interview Questions

### 1. What is a Namespace?

A logical partition in Kubernetes cluster for organizing and isolating resources.

### 2. Why are namespaces used?

For multi-team usage, isolation, and resource organization.

### 3. What happens when a namespace is deleted?

All resources inside it are deleted.

### 4. Can same Pod names exist in different namespaces?

Yes.

### 5. What is the default namespace?

`default`
