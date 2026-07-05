# Kubernetes RBAC Practice

## Objective

Learn how Kubernetes RBAC (Role-Based Access Control) works by creating:

* A custom Service Account
* A Role
* A RoleBinding
* A Deployment that uses the Service Account
* Verify the assigned permissions using `kubectl auth can-i`

---

# Project Structure

```text
rbac-example/
│
├── deployment.yml
├── serviceaccount.yml
├── role.yml
├── rolebinding.yml
└── README.md
```

---

# RBAC Components

## 1. Service Account

A Service Account represents an application running inside the Kubernetes cluster.

Instead of using the default Service Account, we create a custom one.

Apply:

```bash
kubectl apply -f serviceaccount.yml
```

Verify:

```bash
kubectl get sa
```

---

## 2. Role

A Role defines **what actions are allowed** on Kubernetes resources **within a namespace**.

Example permissions in this project:

* Get Pods
* List Pods
* Watch Pods

Apply:

```bash
kubectl apply -f role.yml
```

Verify:

```bash
kubectl get roles
kubectl describe role pod-reader
```

---

## 3. RoleBinding

A RoleBinding connects a Role to a User or Service Account.

In this project:

```text
backend-sa
      │
      ▼
RoleBinding
      │
      ▼
pod-reader Role
```

Apply:

```bash
kubectl apply -f rolebinding.yml
```

Verify:

```bash
kubectl get rolebindings
kubectl describe rolebinding backend-binding
```

---

## 4. Deployment

The Deployment runs Pods using the custom Service Account.

Example:

```yaml
serviceAccountName: backend-sa
```

Apply:

```bash
kubectl apply -f deployment.yml
```

Verify:

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

Expected:

```text
Service Account: backend-sa
```

---

# Verify RBAC Permissions (Recommended Method)

Use `kubectl auth can-i` to verify whether the Service Account has permission to perform a specific action.

### Can get Pods?

```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:backend-sa
```

Expected Output:

```text
yes
```

---

### Can list Pods?

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:backend-sa
```

Expected Output:

```text
yes
```

---

### Can watch Pods?

```bash
kubectl auth can-i watch pods --as=system:serviceaccount:default:backend-sa
```

Expected Output:

```text
yes
```

---

### Can create Pods?

```bash
kubectl auth can-i create pods --as=system:serviceaccount:default:backend-sa
```

Expected Output:

```text
no
```

---

### Can delete Pods?

```bash
kubectl auth can-i delete pods --as=system:serviceaccount:default:backend-sa
```

Expected Output:

```text
no
```

---

# Useful Commands

```bash
kubectl get sa

kubectl get roles

kubectl get rolebindings

kubectl get pods

kubectl describe role pod-reader

kubectl describe rolebinding backend-binding

kubectl describe pod <pod-name>

kubectl auth can-i get pods --as=system:serviceaccount:default:backend-sa

kubectl auth can-i list pods --as=system:serviceaccount:default:backend-sa

kubectl auth can-i watch pods --as=system:serviceaccount:default:backend-sa

kubectl auth can-i create pods --as=system:serviceaccount:default:backend-sa

kubectl auth can-i delete pods --as=system:serviceaccount:default:backend-sa
```

---

# RBAC Workflow

```text
Application (Pod)
        │
        ▼
Service Account
        │
        ▼
RoleBinding
        │
        ▼
Role
        │
        ▼
Permissions
        │
        ▼
Kubernetes API Server
        │
        ▼
Request Allowed / Forbidden
```

---

# Key Learnings

* RBAC is Kubernetes' authorization mechanism.
* Service Accounts provide identities for applications running inside the cluster.
* Roles define permissions within a namespace.
* RoleBindings attach Roles to Service Accounts or Users.
* `kubectl auth can-i` is the quickest and most common way to verify RBAC permissions.
* A Pod must use the intended Service Account (`serviceAccountName`) for the assigned Role to take effect.
* If an action is not permitted by the Role, Kubernetes returns a **Forbidden** response.

