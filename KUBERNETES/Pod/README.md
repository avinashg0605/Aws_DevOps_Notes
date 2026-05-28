# Kubernetes Pods Notes

## What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes.

A Pod wraps one or more containers and provides:

* Shared networking
* Shared storage
* Common lifecycle
* Scheduling as a single unit

Think of a Pod as a logical host for containers.

---

# Key Characteristics of Pods

## 1. Single or Multiple Containers

Most Pods contain **one container**.

Example:

* One Pod → One Nginx container

Sometimes Pods contain **multiple tightly coupled containers**:

* Main application container
* Sidecar container (logging, monitoring, proxy)

---

## 2. Shared Network

All containers inside a Pod:

* Share the same IP address
* Share the same port space
* Communicate using `localhost`

Example:

* App container talks to logging container using `localhost:8080`

---

## 3. Shared Storage

Containers in the same Pod can share volumes.

Example:

```yaml
volumes:
  - name: shared-data
```

Mounted into multiple containers.

---

# Pod Lifecycle

A Pod goes through several phases:

| Phase     | Description                             |
| --------- | --------------------------------------- |
| Pending   | Pod accepted but containers not started |
| Running   | Containers are running                  |
| Succeeded | Containers completed successfully       |
| Failed    | One or more containers failed           |
| Unknown   | Kubernetes cannot determine state       |

---

# Basic Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

---

# Important Fields in Pod Definition

| Field      | Purpose                |
| ---------- | ---------------------- |
| apiVersion | Kubernetes API version |
| kind       | Resource type          |
| metadata   | Pod name, labels       |
| spec       | Desired configuration  |
| containers | List of containers     |
| image      | Container image        |
| ports      | Exposed ports          |

---

# Creating a Pod

```bash
kubectl apply -f pod.yaml
```

---

# Viewing Pods

## List Pods

```bash
kubectl get pods
```

## Detailed Information

```bash
kubectl describe pod nginx-pod
```

## Logs

```bash
kubectl logs nginx-pod
```

---

# Executing Inside a Pod

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

Useful for debugging.

---

# Labels and Selectors

Labels help organize Pods.

Example:

```yaml
metadata:
  labels:
    app: nginx
    env: production
```

Query:

```bash
kubectl get pods -l app=nginx
```

---

# Pod Networking

Each Pod gets:

* Unique IP inside cluster
* Flat networking model

Pods can communicate directly without NAT.

---

# Init Containers

Run before main containers start.

Example uses:

* Database migration
* Waiting for dependencies
* Configuration setup

Example:

```yaml
initContainers:
  - name: init-service
    image: busybox
    command: ['sh', '-c', 'echo Initializing']
```

---

# Multi-Container Pod Pattern

Common patterns:

| Pattern    | Purpose             |
| ---------- | ------------------- |
| Sidecar    | Logging/monitoring  |
| Ambassador | Proxy communication |
| Adapter    | Data transformation |

---

# Pod Restart Policy

```yaml
restartPolicy: Always
```

Options:

* Always
* OnFailure
* Never

---

# Probes in Pods

## Liveness Probe

Checks if app is alive.

## Readiness Probe

Checks if app is ready for traffic.

## Startup Probe

Checks startup completion.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
```

---

# Resource Requests and Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"

  limits:
    memory: "128Mi"
    cpu: "500m"
```

Purpose:

* Prevent resource starvation
* Better scheduling

---

# Volumes in Pods

Volumes provide persistent/shared storage.

Common types:

* emptyDir
* hostPath
* configMap
* secret
* persistentVolumeClaim

Example:

```yaml
volumes:
  - name: app-storage
    emptyDir: {}
```

---

# Why Pods are Ephemeral

Pods are temporary:

* Can be recreated anytime
* IP may change
* Should not store critical local state

Use:

* Deployments
* StatefulSets
* Persistent Volumes

for production workloads.

---

# Controllers That Manage Pods

Usually Pods are NOT created directly in production.

Controllers create/manage Pods:

| Controller  | Purpose                 |
| ----------- | ----------------------- |
| Deployment  | Stateless apps          |
| StatefulSet | Stateful apps           |
| DaemonSet   | One Pod per node        |
| Job         | Run-to-completion tasks |
| CronJob     | Scheduled tasks         |

---

# Common kubectl Commands

| Command                       | Description  |
| ----------------------------- | ------------ |
| kubectl get pods              | List Pods    |
| kubectl describe pod NAME     | Pod details  |
| kubectl logs NAME             | View logs    |
| kubectl delete pod NAME       | Delete Pod   |
| kubectl exec -it NAME -- bash | Shell access |

---

# Example: Pod with Labels and Resources

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: app-pod
  labels:
    app: myapp

spec:
  containers:
    - name: app-container
      image: nginx

      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"

        limits:
          memory: "128Mi"
          cpu: "500m"

      ports:
        - containerPort: 80
```

---

# Best Practices

## Do

* Use Deployments instead of standalone Pods
* Add readiness/liveness probes
* Set resource limits
* Use labels properly
* Keep containers lightweight

## Avoid

* Storing important data inside Pods
* Running many unrelated containers in one Pod
* Creating Pods manually in production

---

# Quick Revision

## Pod =

* Smallest Kubernetes unit
* One or more containers
* Shared IP + storage
* Ephemeral

## Important Concepts

* Labels
* Probes
* Volumes
* Resource limits
* Init containers
* Sidecars

---

# Interview Questions

### 1. What is a Pod?

Smallest deployable Kubernetes unit containing one or more containers.

### 2. Can multiple containers run in a Pod?

Yes, if tightly coupled.

### 3. Do containers in a Pod share network?

Yes, same IP and localhost communication.

### 4. Why are Pods ephemeral?

They can be destroyed/recreated anytime by Kubernetes.

### 5. Difference between Pod and Container?

Container runs application; Pod wraps containers with networking/storage abstraction.
