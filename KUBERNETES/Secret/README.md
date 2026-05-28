# Kubernetes Secrets Notes

# Table of Contents

1. What is a Secret?
2. Why Secrets are Needed
3. Secret Architecture
4. Types of Kubernetes Secrets
5. Creating Secrets
6. Secret YAML
7. Base64 Encoding
8. Using Secrets in Pods
9. Secrets as Environment Variables
10. Secrets as Volume Mounts
11. Docker Registry Secrets
12. Updating Secrets
13. Immutable Secrets
14. Security Best Practices
15. Troubleshooting
16. Interview Questions
17. Scenario-Based Questions
18. Production Tips
19. Quick Revision

---

# 1. What is a Secret?

A **Secret** in Kubernetes is an object used to store sensitive information securely.

Examples:

* Passwords
* API keys
* Tokens
* SSH keys
* TLS certificates

---

# 2. Why Secrets are Needed

Without Secrets:

* Credentials may be hardcoded in application code
* Sensitive data exposed in container images
* Security risk increases

Secrets provide:

* Centralized secure storage
* Better security management
* Controlled access

---

# Example

Instead of:

```text id="w7s9kj"
DB_PASSWORD=admin123
```

inside application code,

store password in Secret.

---

# 3. Secret Architecture

```text id="z1n6qc"
Secret
   ↓
Pod
   ↓
Container
```

Secrets can be injected into containers using:

* Environment variables
* Mounted volumes

---

# 4. Types of Kubernetes Secrets

| Type                           | Purpose                     |
| ------------------------------ | --------------------------- |
| Opaque                         | Generic secret              |
| kubernetes.io/dockerconfigjson | Docker registry credentials |
| kubernetes.io/tls              | TLS certificates            |
| kubernetes.io/basic-auth       | Username/password           |
| kubernetes.io/ssh-auth         | SSH credentials             |

---

# Default Type

```text id="s5x2up"
Opaque
```

Most commonly used.

---

# 5. Creating Secrets

# Method 1: Imperative Command

```bash id="j4t7dy"
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=admin123
```

---

# Verify Secret

```bash id="f8k3xp"
kubectl get secrets
```

---

# Describe Secret

```bash id="b6v1nz"
kubectl describe secret db-secret
```

---

# 6. Secret YAML

# Important

Secret values must be Base64 encoded.

---

# Example YAML

```yaml id="m7c2rw"
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

type: Opaque

data:
  username: YWRtaW4=
  password: YWRtaW4xMjM=
```

Apply:

```bash id="p2n8yk"
kubectl apply -f secret.yaml
```

---

# 7. Base64 Encoding

## Encode Value

```bash id="d8w5ts"
echo -n 'admin123' | base64
```

---

# Decode Value

```bash id="n1q7vm"
echo 'YWRtaW4xMjM=' | base64 --decode
```

---

# Important Security Note

Base64 is:

* NOT encryption
* Only encoding

---

# 8. Using Secrets in Pods

Secrets can be consumed by containers.

Methods:

1. Environment variables
2. Volume mounts

---

# 9. Secrets as Environment Variables

## Example

```yaml id="q5x8pe"
apiVersion: v1
kind: Pod

metadata:
  name: secret-pod

spec:
  containers:
    - name: nginx
      image: nginx

      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

---

# Verify Inside Pod

```bash id="y4v9rw"
kubectl exec -it secret-pod -- env
```

---

# 10. Secrets as Volume Mounts

Secrets become files inside container.

---

# Example

```yaml id="h8m3zt"
apiVersion: v1
kind: Pod

metadata:
  name: secret-volume-pod

spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets

  volumes:
    - name: secret-volume
      secret:
        secretName: db-secret
```

---

# Result Inside Container

```text id="u0r5nc"
/etc/secrets/username
/etc/secrets/password
```

Each key becomes a file.

---

# 11. Docker Registry Secrets

Used for private container registries.

---

# Create Docker Secret

```bash id="t2m9qk"
kubectl create secret docker-registry reg-secret \
--docker-server=docker.io \
--docker-username=myuser \
--docker-password=mypassword \
--docker-email=myemail@example.com
```

---

# Use in Pod

```yaml id="g4p1ws"
imagePullSecrets:
  - name: reg-secret
```

---

# 12. Updating Secrets

## Edit Secret

```bash id="e5v7nd"
kubectl edit secret db-secret
```

---

# Important Behavior

Environment variables:

* Require Pod restart

Volume-mounted secrets:

* Auto-refresh after delay

---

# Restart Deployment

```bash id="x7n3zb"
kubectl rollout restart deployment app
```

---

# 13. Immutable Secrets

Prevent accidental modification.

Example:

```yaml id="v1q9yr"
immutable: true
```

Benefits:

* Better performance
* Better security
* Reduced accidental updates

---

# 14. Security Best Practices

# Do

* Use Secrets for credentials
* Enable etcd encryption
* Use RBAC restrictions
* Rotate secrets regularly
* Use external secret managers

---

# Avoid

* Hardcoding passwords
* Storing secrets in Git
* Logging secret values
* Sharing secrets across namespaces unnecessarily

---

# Recommended Tools

* HashiCorp Vault
* AWS Secrets Manager
* Azure Key Vault
* External Secrets Operator

---

# 15. Troubleshooting

# Problem 1: Secret Not Found

Check:

```bash id="o9m2xt"
kubectl get secrets
```

Verify:

* Secret name
* Namespace

---

# Problem 2: Pod Fails to Start

Check Pod events:

```bash id="k8w1vr"
kubectl describe pod POD_NAME
```

Possible causes:

* Missing Secret
* Wrong key name

---

# Problem 3: Wrong Secret Value

Decode secret:

```bash id="c2x7mp"
kubectl get secret db-secret -o yaml
```

Verify Base64 encoding.

---

# Problem 4: Application Using Old Secret

Environment variables require Pod restart.

Restart:

```bash id="q4n8tw"
kubectl rollout restart deployment app
```

---

# 16. Interview Questions

# Beginner Level

### 1. What is Kubernetes Secret?

Object used to store sensitive data securely.

---

### 2. Why use Secrets instead of ConfigMaps?

Secrets are meant for sensitive data.

---

### 3. Are Secret values encrypted?

By default:

* Base64 encoded
* Not fully encrypted

Encryption at rest must be enabled separately.

---

### 4. How can Secrets be consumed?

* Environment variables
* Volume mounts

---

### 5. What is Opaque Secret?

Generic secret type.

---

# Intermediate Level

### 6. Difference between Secret and ConfigMap?

| ConfigMap     | Secret         |
| ------------- | -------------- |
| Non-sensitive | Sensitive      |
| Plain text    | Base64 encoded |

---

### 7. What is imagePullSecret?

Secret used for private container registry authentication.

---

### 8. Where are Secrets stored?

Inside etcd.

---

### 9. Can Secrets be namespaced?

Yes.

---

### 10. What is immutable Secret?

Secret that cannot be modified after creation.

---

# Advanced Level

### 11. How do you secure Secrets in production?

* Encrypt etcd
* Use RBAC
* Rotate credentials
* Use external secret managers

---

### 12. What happens if Secret is deleted?

Dependent Pods may fail.

---

### 13. Why is Base64 not secure?

It is encoding, not encryption.

---

### 14. Can Pods access Secrets from other namespaces?

No by default.

---

### 15. How do Secrets improve CI/CD?

Avoid hardcoded credentials in pipelines.

---

# 17. Scenario-Based Questions

# Scenario 1

## Question

Developer hardcoded DB password inside application image.

## Answer

Move credentials to Kubernetes Secrets.

---

# Scenario 2

## Question

Application cannot access private Docker image.

## Answer

Use imagePullSecrets.

---

# Scenario 3

## Question

Secret updated but application still uses old value.

## Answer

Restart Pods if using environment variables.

---

# Scenario 4

## Question

Need TLS certificates for HTTPS ingress.

## Answer

Use TLS Secret.

---

# Scenario 5

## Question

Security team asks for secret rotation.

## Answer

Update Secrets and restart workloads.

---

# Scenario 6

## Question

Need centralized secret management across clusters.

## Answer

Use external secret managers like Vault.

---

# Scenario 7

## Question

A Pod cannot find Secret.

## Answer

Check:

* Namespace
* Secret name
* Key names
* RBAC permissions

---

# 18. Production Tips

# Enable etcd Encryption

Critical for production security.

---

# Use Least Privilege Access

Restrict access using:

* RBAC
* Service accounts

---

# Rotate Secrets Regularly

Examples:

* DB passwords
* API tokens
* Certificates

---

# Avoid Long-Lived Credentials

Use:

* Temporary tokens
* IAM roles

---

# Integrate External Secret Managers

Recommended for enterprise environments.

---

# 19. Quick Revision

## Secret =

* Stores sensitive data
* Key-value object
* Base64 encoded

---

# Ways to Use

* Environment variables
* Volume mounts
* imagePullSecrets

---

# Important Concepts

* Opaque Secret
* TLS Secret
* Docker registry Secret
* Base64 encoding
* Secret rotation

---

# Important Commands

```bash id="m6q9vx"
kubectl get secrets
kubectl describe secret NAME
kubectl edit secret NAME
kubectl delete secret NAME
```

---

# Most Important Interview Point

Secrets are Base64 encoded by default, NOT encrypted.

For production:

* Enable etcd encryption
* Use RBAC
* Rotate credentials regularly
