# Kubernetes Deployment Notes

## What is a Deployment?

A **Deployment** in Kubernetes is a controller that manages Pods and ReplicaSets.

Deployment provides:

* Declarative updates
* Scaling
* Self-healing
* Rolling updates
* Rollbacks

---

# Why Deployments are Needed

Pods alone are not reliable because:

* They can fail
* They are ephemeral
* Manual management is difficult

Deployments automate Pod management.

---

# Deployment Architecture

```text id="5owxgp"
Deployment → ReplicaSet → Pods
```

---

# Features of Deployments

* Creates and manages Pods
* Maintains desired number of replicas
* Replaces failed Pods
* Supports rolling updates
* Enables rollback to previous version

---

# Basic Deployment YAML

```yaml id="mqy8r7"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

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

# Important Fields

| Field      | Purpose                  |
| ---------- | ------------------------ |
| replicas   | Number of Pod replicas   |
| selector   | Identifies matching Pods |
| template   | Pod template             |
| containers | Container configuration  |

---

# Creating a Deployment

```bash id="k2u3x7"
kubectl apply -f deployment.yaml
```

---

# Viewing Deployments

## List Deployments

```bash id="3xtk8j"
kubectl get deployments
```

OR

```bash id="z7e4mv"
kubectl get deploy
```

---

## Describe Deployment

```bash id="7zv4wy"
kubectl describe deployment nginx-deployment
```

---

# Viewing ReplicaSets

```bash id="9rm0dp"
kubectl get rs
```

---

# Scaling Deployments

## Scale Manually

```bash id="5yr3tu"
kubectl scale deployment nginx-deployment --replicas=5
```

---

# Rolling Updates

Deployment supports zero-downtime updates.

Example:

```bash id="3u9s4m"
kubectl set image deployment/nginx-deployment \
nginx=nginx:1.25
```

Kubernetes gradually replaces old Pods with new ones.

---

# Deployment Strategies

| Strategy      | Description                      |
| ------------- | -------------------------------- |
| RollingUpdate | Default gradual update           |
| Recreate      | Deletes old Pods before new Pods |

---

# Rollback Deployment

View rollout history:

```bash id="4bq7w2"
kubectl rollout history deployment nginx-deployment
```

Rollback:

```bash id="x9m5p1"
kubectl rollout undo deployment nginx-deployment
```

---

# Restart Deployment

```bash id="v8z0ep"
kubectl rollout restart deployment nginx-deployment
```

---

# Pause and Resume Deployment

Pause:

```bash id="m7v3s1"
kubectl rollout pause deployment nginx-deployment
```

Resume:

```bash id="f0c2rx"
kubectl rollout resume deployment nginx-deployment
```

---

# Self-Healing

If a Pod crashes:

* Deployment automatically recreates it

This ensures desired state is maintained.

---

# Labels and Selectors

Deployment uses labels to manage Pods.

Example:

```yaml id="2n1d8k"
labels:
  app: nginx
```

Selector:

```yaml id="4q5r8w"
selector:
  matchLabels:
    app: nginx
```

---

# Updating Deployments

Edit deployment:

```bash id="m2r8q5"
kubectl edit deployment nginx-deployment
```

OR update YAML and apply again.

---

# Deleting Deployment

```bash id="k8x4tw"
kubectl delete deployment nginx-deployment
```

---

# Deployment vs ReplicaSet

| Feature         | Deployment | ReplicaSet      |
| --------------- | ---------- | --------------- |
| Rolling Updates | Yes        | No              |
| Rollbacks       | Yes        | No              |
| Pod Management  | Advanced   | Basic           |
| Recommended     | Yes        | Rarely directly |

---

# Best Practices

## Do

* Use Deployments for stateless apps
* Set resource limits
* Use rolling updates
* Add probes

## Avoid

* Managing ReplicaSets manually
* Using latest tag in production
* Running single replica in production

---

# Common kubectl Commands

| Command                  | Description         |
| ------------------------ | ------------------- |
| kubectl get deploy       | List deployments    |
| kubectl scale deployment | Scale replicas      |
| kubectl rollout status   | Check rollout       |
| kubectl rollout undo     | Rollback deployment |

---

# Quick Revision

## Deployment =

* Manages Pods
* Ensures desired replicas
* Supports updates
* Provides rollback

---

# Important Concepts

* ReplicaSet
* Rolling updates
* Rollbacks
* Scaling
* Self-healing

---

# Interview Questions

### 1. What is a Deployment?

A Kubernetes controller that manages Pods and ReplicaSets.

### 2. Why use Deployments instead of Pods?

Deployments provide scaling, updates, rollback, and self-healing.

### 3. What is rolling update?

Gradual replacement of old Pods with new Pods without downtime.

### 4. What is ReplicaSet?

Controller ensuring desired number of Pod replicas.

### 5. Can Deployment rollback updates?

Yes.
