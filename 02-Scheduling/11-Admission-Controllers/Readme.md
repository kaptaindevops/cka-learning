# Admission Controllers in Kubernetes

When we run commands using **kubectl**, such as creating a Pod or Deployment, the request is sent to the **Kubernetes API Server**.

The API Server processes the request through several stages before the object is created and stored in **etcd**.

The typical flow looks like this:

1. The user sends a request using `kubectl`
2. The request reaches the **API Server**
3. The request is **authenticated**
4. The request is **authorized**
5. The request passes through **Admission Controllers**
6. If approved, the object is created and stored in **etcd**

Admission Controllers are responsible for **validating and modifying requests before they are persisted in the cluster**.

---

# Authentication

The first step is authentication.

Authentication verifies **who is making the request**.

In Kubernetes, this is often done using **certificates**.

When you use `kubectl`, the credentials are stored in the **kubeconfig file**.  
The API server reads these credentials and verifies the identity of the user.

If the user cannot be authenticated, the request is rejected immediately.

---

# Authorization

Once the user identity is verified, Kubernetes checks whether the user has permission to perform the requested action.

This is handled through **authorization mechanisms**, most commonly **Role-Based Access Control (RBAC)**.

For example, a role may allow a developer to:

- Create Pods
- List Pods
- Update Pods
- Delete Pods

If a user tries to perform an action outside their permissions, the request is denied.

RBAC focuses on **API-level permissions**, such as:

- What resources a user can access
- Which actions they can perform
- Which namespaces they can operate in

However, RBAC does not inspect the **content of the request itself**.

---

# Limitations of RBAC

RBAC controls **who can perform operations**, but it cannot enforce deeper configuration rules.

For example, RBAC cannot enforce rules such as:

- Allow only images from an internal container registry
- Prevent usage of the `latest` image tag
- Ensure containers do not run as root
- Require certain labels on all resources
- Restrict specific container capabilities

These types of checks require inspecting the **object configuration itself**.

This is where **Admission Controllers** become useful.

---

# What Are Admission Controllers

Admission Controllers are plugins that intercept requests **after authentication and authorization but before the object is stored in etcd**.

They allow Kubernetes to:

- Validate incoming requests
- Modify requests before they are applied
- Enforce security policies
- Apply default values to resources

In simple terms, they act as **gatekeepers for cluster configuration**.

---

# Examples of Built-in Admission Controllers

Kubernetes includes several built-in admission controllers.

Some commonly used ones include:

### AlwaysPullImages

Ensures that container images are always pulled from the registry whenever a Pod is created.

This helps ensure the latest version of the image is used.

---

### DefaultStorageClass

Automatically assigns a default storage class to PersistentVolumeClaims if none is specified.

---

### EventRateLimit

Controls the rate of events sent to the API Server to prevent it from being overloaded.

---

### NamespaceExists

Ensures that a resource cannot be created in a namespace that does not exist.

If a request is made to a non-existing namespace, the request is rejected.

---

# Example: Namespace Validation

Suppose a user attempts to create a Pod in a namespace called **blue** that does not exist.

Example command:

```
kubectl run web --image=nginx -n blue
```

The request follows this process:

1. The API server authenticates the user.
2. Authorization checks if the user can create Pods.
3. The request reaches the **NamespaceExists admission controller**.
4. The controller verifies whether the namespace exists.
5. If the namespace does not exist, the request is rejected.

The user receives an error indicating that the namespace cannot be found.

---

# Automatically Creating Namespaces

Another admission controller called **NamespaceAutoProvision** can automatically create namespaces if they do not exist.

If enabled, the same request would result in:

1. Namespace being created automatically
2. Pod creation continuing successfully

However, this controller is no longer commonly used and has been replaced by newer mechanisms.

---

# Viewing Enabled Admission Controllers

You can view enabled admission controllers by checking the API server configuration.

Example command:

```bash
kube-apiserver --help | grep enable-admission-plugins
```

In kubeadm clusters where the API server runs as a Pod, you may need to inspect the API server manifest.

Example location:

```
/etc/kubernetes/manifests/kube-apiserver.yaml
```

Inside the file, look for:

```
--enable-admission-plugins
```

This flag lists all admission controllers currently enabled.

---

# Enabling a New Admission Controller

To enable an admission controller, modify the API server configuration.

Example:

```
--enable-admission-plugins=NamespaceLifecycle,LimitRanger
```

After updating the configuration, the API server must restart for the changes to take effect.

In kubeadm-based clusters, the API server restarts automatically when the manifest file changes.

---

# Disabling an Admission Controller

Admission controllers can also be disabled.

Example:

```
--disable-admission-plugins=NamespaceLifecycle
```

This prevents the selected controller from running.

---

# NamespaceLifecycle Admission Controller

The older controllers **NamespaceExists** and **NamespaceAutoProvision** have been deprecated.

They were replaced by the **NamespaceLifecycle admission controller**.

This controller performs several important checks:

- Rejects requests to namespaces that do not exist
- Prevents deletion of critical namespaces such as:

  - `default`
  - `kube-system`
  - `kube-public`

This ensures system namespaces remain protected.

---

# Summary

Admission Controllers provide an additional layer of control in Kubernetes.

The request flow typically follows this order:

1. Authentication
2. Authorization
3. Admission Controllers
4. Object creation in etcd

Admission Controllers help enforce cluster policies such as:

- Security restrictions
- Default configurations
- Resource validation

They allow Kubernetes administrators to ensure that workloads follow defined operational and security rules before being deployed in the cluster.
