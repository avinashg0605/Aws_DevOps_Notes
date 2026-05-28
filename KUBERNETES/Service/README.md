# Kubernetes Services Notes

## What is a Service in Kubernetes?

A **Service** in Kubernetes is an abstraction that exposes a set of Pods as a network service.

Since Pods are ephemeral and their IP addresses can change, Services provide:

* Stable IP address
* Stable DNS name
* Load balancing
* Service discovery

---

# Why Services are Needed

Pods:

* Can die and restart
* Get new IP addresses
* Are temporary

Without Services:

* Applications cannot reliably communicate

Service solves this by providing a stable endpoint.

---

# How Services Work

Services use:

* Labels
* Selectors

to identify target Pods.

Example:

```yaml id="m7u2eq"
selector:
  app: nginx
```

The Service forwards traffic to Pods matching the label.

---

# Types of Kubernetes Services

| Type             | Purpose                                   |
| ---------------- | ----------------------------------------- |
| ClusterIP        | Internal communication inside cluster     |
| NodePort         | Expose app on node IP                     |
| LoadBalancer     | External access using cloud load balancer |
| ExternalName     | Maps Service to external DNS              |
| Headless Service | Direct Pod access without cluster IP      |

---

# 1. ClusterIP Service

Default Service type.

Used for:

* Internal communication between microservices

Accessible only inside cluster.

---

## ClusterIP Example

```yaml id="z8t1lf"
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

# 2. NodePort Service

Exposes application on:

* Node IP
* Static port

Accessible externally using:

```text id="dgrg4j"
<NodeIP>:<NodePort>
```

---

## NodePort Example

```yaml id="0wy6a4"
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

---

# 3. LoadBalancer Service

Used in cloud environments:

* AWS
* Azure
* GCP

Creates external cloud load balancer automatically.

---

## LoadBalancer Example

```yaml id="n4v0jp"
apiVersion: v1
kind: Service

metadata:
  name: nginx-loadbalancer

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

---

# 4. ExternalName Service

Maps Kubernetes Service to external DNS.

Example:

* External database
* Third-party APIs

---

## ExternalName Example

```yaml id="uxb5k6"
apiVersion: v1
kind: Service

metadata:
  name: external-service

spec:
  type: ExternalName
  externalName: example.com
```

---

# 5. Headless Service

Created using:

```yaml id="tb1o7e"
clusterIP: None
```

Purpose:

* Direct Pod-to-Pod communication
* Stateful applications

Used with:

* StatefulSets
* Databases

---

# Service Discovery

Kubernetes provides built-in DNS.

Pods can access services using:

```text id="vqutwd"
<service-name>
```

Example:

```text id="6g5y92"
http://nginx-service
```

---

# Labels and Selectors

Service selects Pods using labels.

Pod Example:

```yaml id="r1w3vf"
metadata:
  labels:
    app: nginx
```

Service Example:

```yaml id="vl04y9"
selector:
  app: nginx
```

---

# Port Definitions

| Field      | Meaning            |
| ---------- | ------------------ |
| port       | Service port       |
| targetPort | Container port     |
| nodePort   | External node port |

---

# Service YAML Example

```yaml id="3j9r6k"
apiVersion: v1
kind: Service

metadata:
  name: my-service

spec:
  selector:
    app: myapp

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

  type: ClusterIP
```

---

# Creating a Service

```bash id="xwrzmn"
kubectl apply -f service.yaml
```

---

# Viewing Services

## List Services

```bash id="n96t2f"
kubectl get svc
```

OR

```bash id="30y5nm"
kubectl get services
```

---

## Describe Service

```bash id="jsg6y5"
kubectl describe svc nginx-service
```

---

# Exposing Deployments as Services

Automatically create Service:

```bash id="ffl5a3"
kubectl expose deployment nginx \
  --type=NodePort \
  --port=80
```

---

# Endpoint Object

Service creates endpoints automatically.

Endpoints contain:

* Pod IPs
* Target ports

View endpoints:

```bash id="t8r17p"
kubectl get endpoints
```

---

# Service Networking Flow

```text id="jlvbif"
Client → Service → Pod
```

Kubernetes performs:

* Load balancing
* Traffic routing

---

# kube-proxy

`kube-proxy` runs on every node.

Responsibilities:

* Implements Service networking
* Maintains iptables/ipvs rules
* Routes traffic to Pods

---

# Headless vs Normal Service

| Feature        | Normal Service | Headless Service |
| -------------- | -------------- | ---------------- |
| Cluster IP     | Yes            | No               |
| Load Balancing | Yes            | No               |
| DNS Returns    | Service IP     | Pod IPs          |

---

# Best Practices

## Do

* Use ClusterIP for internal apps
* Use LoadBalancer for production external access
* Use labels consistently
* Keep selectors simple

## Avoid

* Hardcoding Pod IPs
* Exposing unnecessary NodePorts
* Using NodePort in production cloud setups

---

# Common kubectl Commands

| Command                   | Description     |
| ------------------------- | --------------- |
| kubectl get svc           | List Services   |
| kubectl describe svc NAME | Service details |
| kubectl expose deployment | Create Service  |
| kubectl delete svc NAME   | Delete Service  |

---

# Quick Revision

## Service =

* Stable network endpoint
* Exposes Pods
* Enables communication
* Provides load balancing

---

# Important Concepts

* ClusterIP
* NodePort
* LoadBalancer
* Selectors
* DNS
* kube-proxy
* Endpoints

---

# Interview Questions

### 1. Why do we need Services in Kubernetes?

Pods are ephemeral and change IPs. Services provide stable communication.

### 2. What is the default Service type?

ClusterIP.

### 3. Difference between port and targetPort?

* `port` → Service port
* `targetPort` → Container port

### 4. What is Headless Service?

A Service without cluster IP used for direct Pod access.

### 5. Difference between NodePort and LoadBalancer?

* NodePort exposes app using node IP and port.
* LoadBalancer creates external cloud load balancer.
