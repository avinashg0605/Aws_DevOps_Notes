# Kubernetes ConfigMaps Notes

# Table of Contents

1. What is ConfigMap?
2. Why ConfigMaps are Needed
3. ConfigMap Architecture
4. Types of Configuration
5. Creating ConfigMaps
6. ConfigMap YAML
7. Using ConfigMaps in Pods
8. ConfigMaps as Environment Variables
9. ConfigMaps as Volume Mounts
10. ConfigMaps with Multiple Keys
11. Updating ConfigMaps
12. Immutable ConfigMaps
13. ConfigMap Best Practices
14. Troubleshooting
15. Interview Questions
16. Scenario-Based Questions
17. Production Tips
18. Quick Revision

---

# 1. What is ConfigMap?

A **ConfigMap** is a Kubernetes object used to store non-sensitive configuration data as key-value pairs.

Used for:

* Environment variables
* Application configuration
* URLs
* Ports
* Feature flags

---

# 2. Why ConfigMaps are Needed

Without ConfigMaps:

* Configuration is hardcoded inside container image
* Any config change requires rebuilding image

ConfigMaps solve this by separating:

* Application code
* Configuration

---

# Example

Instead of:

```text id="n0e5qy"
DB_HOST=localhost
```

inside application code,

store it in ConfigMap.

---

# 3. ConfigMap Architecture

```text id="j8v2mw"
ConfigMap
    ↓
Pod
    ↓
Container
```

ConfigMap data can be injected into Pods using:

* Environment variables
* Volume mounts
* Command arguments

---

# 4. Types of Configuration

ConfigMaps store:

* Application settings
* Database URLs
* API endpoints
* Logging levels
* Feature toggles

---

# NOT Suitable For

Do NOT store:

* Passwords
* Tokens
* Secrets

Use Kubernetes Secrets instead.

---

# 5. Creating ConfigMaps

# Method 1: Imperative Command

```bash id="v1m4r9"
kubectl create configmap app-config \
--from-literal=APP_MODE=production
```

---

# Verify

```bash id="p7x3dc"
kubectl get configmaps
```

---

# Describe ConfigMap

```bash id="n4t9sk"
kubectl describe configmap app-config
```

---

# 6. ConfigMap YAML

```yaml id="x8f2jk"
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_MODE: production
  APP_COLOR: blue
  LOG_LEVEL: debug
```

Apply:

```bash id="r9m2cq"
kubectl apply -f configmap.yaml
```

---

# 7. Using ConfigMaps in Pods

ConfigMaps can be consumed by Pods.

Methods:

1. Environment Variables
2. Volume Mounts
3. Command-line arguments

---

# 8. ConfigMaps as Environment Variables

## Example Pod

```yaml id="d3n7qx"
apiVersion: v1
kind: Pod

metadata:
  name: configmap-pod

spec:
  containers:
    - name: nginx
      image: nginx

      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
```

---

# Verify Inside Pod

```bash id="u1z5nm"
kubectl exec -it configmap-pod -- env
```

---

# 9. ConfigMaps as Volume Mounts

ConfigMap values become files inside container.

---

# Example

```yaml id="w7b9qe"
apiVersion: v1
kind: Pod

metadata:
  name: volume-configmap-pod

spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: config-volume
          mountPath: /etc/config

  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

---

# Result Inside Container

```text id="o5p8tw"
/etc/config/APP_MODE
/etc/config/APP_COLOR
```

Each key becomes a file.

---

# 10. ConfigMaps with Multiple Keys

Example:

```yaml id="m3k8ve"
data:
  database_url: mysql-service
  redis_host: redis-service
  app_mode: production
```

Useful for centralized application configuration.

---

# 11. Updating ConfigMaps

## Edit ConfigMap

```bash id="f8x1vn"
kubectl edit configmap app-config
```

---

# Important Behavior

Environment variables:

* Require Pod restart

Volume mounts:

* Update automatically after short delay

---

# Restart Deployment

```bash id="h2q5st"
kubectl rollout restart deployment nginx
```

---

# 12. Immutable ConfigMaps

Prevent accidental updates.

Example:

```yaml id="k6z4wc"
immutable: true
```

Benefits:

* Better performance
* Increased safety

---

# 13. ConfigMap Best Practices

## Do

* Separate config from code
* Use meaningful keys
* Use versioned configs
* Store only non-sensitive data

---

## Avoid

* Storing passwords
* Large configuration files
* Hardcoding configs in images

---

# 14. Troubleshooting

# Problem 1: Environment Variable Missing

Check:

* ConfigMap exists
* Key name correct
* Namespace correct

---

# Verify

```bash id="v7r3bp"
kubectl get cm
kubectl describe cm app-config
```

---

# Problem 2: Pod Not Starting

Check events:

```bash id="z2m8cq"
kubectl describe pod POD_NAME
```

Possible cause:

* ConfigMap not found

---

# Problem 3: Updated Config Not Reflecting

Environment variables require Pod restart.

Restart:

```bash id="n5v1dy"
kubectl rollout restart deployment app
```

---

# Problem 4: Volume Files Missing

Verify:

* Volume mount path
* ConfigMap name
* Key names

---

# 15. Interview Questions

# Beginner Level

### 1. What is ConfigMap?

Stores non-sensitive configuration data.

---

### 2. Why use ConfigMaps?

To separate configuration from application code.

---

### 3. How can ConfigMaps be consumed?

* Environment variables
* Volumes
* Command arguments

---

### 4. Can ConfigMaps store secrets?

No.

---

### 5. What format does ConfigMap use?

Key-value pairs.

---

# Intermediate Level

### 6. Difference between ConfigMap and Secret?

ConfigMap:

* Non-sensitive data

Secret:

* Sensitive data

---

### 7. What happens when ConfigMap is updated?

Environment variables:

* Need restart

Volume mounts:

* Auto-refresh

---

### 8. Can multiple Pods use same ConfigMap?

Yes.

---

### 9. What is immutable ConfigMap?

ConfigMap that cannot be modified after creation.

---

### 10. Where are ConfigMaps stored?

Inside etcd.

---

# Advanced Level

### 11. Why are ConfigMaps important in microservices?

Centralized configuration management.

---

### 12. How do ConfigMaps improve CI/CD?

Config changes without rebuilding images.

---

### 13. Can ConfigMaps be namespaced?

Yes.

---

### 14. What happens if ConfigMap is deleted?

Pods depending on it may fail.

---

### 15. Best practice for production configuration?

Use versioned ConfigMaps with Deployments.

---

# 16. Scenario-Based Questions

# Scenario 1

## Question

Application environment changed from dev to prod without rebuilding image. How?

## Answer

Use ConfigMaps for environment-specific configuration.

---

# Scenario 2

## Question

Application cannot read environment variable.

## Answer

Check:

* ConfigMap exists
* Key name
* Pod namespace
* Environment mapping

---

# Scenario 3

## Question

Need same configuration across multiple Pods.

## Answer

Use shared ConfigMap.

---

# Scenario 4

## Question

Configuration changed but application still using old value.

## Answer

Restart Pods if using environment variables.

---

# Scenario 5

## Question

Need centralized logging configuration for all applications.

## Answer

Use common ConfigMap mounted across Pods.

---

# Scenario 6

## Question

Developer accidentally stored passwords in ConfigMap.

## Answer

Move sensitive data to Secrets immediately.

---

# Scenario 7

## Question

Need application-specific config files inside container.

## Answer

Mount ConfigMap as volume.

---

# 17. Production Tips

# Use Versioned ConfigMaps

Example:

```text id="v8n1xt"
app-config-v1
app-config-v2
```

Safer rollbacks.

---

# Avoid Massive ConfigMaps

Large configs:

* Hard to manage
* Hard to debug

---

# Use Immutable ConfigMaps

Improves:

* Stability
* API server performance

---

# Combine with Deployments

Best practice:

```text id="s7r2qp"
Deployment + ConfigMap
```

---

# Monitor Configuration Changes

Track:

* Who changed config
* When config changed

---

# 18. Quick Revision

## ConfigMap =

* Stores non-sensitive config
* Key-value storage
* Separate config from code

---

# Ways to Use

* Environment variables
* Volumes
* Commands

---

# Important Concepts

* Key-value pairs
* Mounted files
* Config updates
* Immutable configs

---

# Important Commands

```bash id="f0n8xe"
kubectl get cm
kubectl describe cm app-config
kubectl edit cm app-config
kubectl delete cm app-config
```

---

# Most Important Interview Point

ConfigMaps are used for non-sensitive configuration.

Secrets are used for sensitive data.
