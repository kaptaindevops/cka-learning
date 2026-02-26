# Kubernetes Namespaces – Organizing and Isolating Resources

## Understanding Namespaces with a Simple Example

Imagine two companies working inside the same office building.

Both companies have a project called **Phoenix**.

Inside each company:

- Employees simply refer to it as "Phoenix".
- Everyone clearly understands which project they are talking about.

However, someone outside the company must specify:

- Phoenix from Company A
- Phoenix from Company B

Each company:

- Has its own team members
- Has its own internal policies
- Has its own budget and resource limits
- Operates independently

In Kubernetes, a **namespace** works in a similar way.

A namespace creates a logical boundary inside a cluster.  
Resources inside one namespace can use simple names, but when referring to resources in another namespace, they must use fully qualified names.

---

# What Is a Namespace in Kubernetes?

A namespace is a way to divide a Kubernetes cluster into multiple virtual environments.

Every resource you create — such as Pods, Services, or Deployments — exists inside a namespace.

If you do not specify one, the resource is created inside the **default namespace**.

---

# Default Namespaces in Kubernetes

When a cluster is created, Kubernetes automatically creates the following namespaces:

### default
Used for user-created workloads if no namespace is specified.

### kube-system
Contains internal Kubernetes components such as DNS and networking services.  
This namespace protects critical system components from accidental modification.

### kube-public
Used for resources that should be accessible cluster-wide.

---

# Why Use Namespaces?

In small environments or learning setups, using the default namespace is sufficient.

However, in production environments, namespaces are extremely important.

Common use cases include:

- Separating development and production workloads
- Isolating different teams in the same cluster
- Applying different access control policies
- Limiting resource consumption per environment

---

# Namespace Isolation in Practice

Assume you are running two environments in the same cluster:

- testing
- production

You can create separate namespaces:

- `testing`
- `production`

This ensures:

- Developers working in testing cannot accidentally modify production.
- Resource limits can be enforced separately.
- Security rules can differ between environments.

---

# Communication Inside and Across Namespaces

## Communication Within the Same Namespace

Resources inside the same namespace can refer to each other using simple names.

Example:

A frontend Pod connects to a backend Service using:

```
backend-service
```

---

## Communication Across Namespaces

If the backend Service exists in another namespace (for example, `testing`), the frontend must use the full DNS name:

```
backend-service.testing.svc.cluster.local
```

Breakdown of the DNS format:

- backend-service → Service name
- testing → Namespace
- svc → Service subdomain
- cluster.local → Default cluster domain

Kubernetes automatically creates this DNS entry when a Service is created.

---

# Working with Namespaces Using kubectl

## List Pods in the Default Namespace

```
kubectl get pods
```

---

## List Pods in Another Namespace

```
kubectl get pods -n production
```

---

## Create a Resource in a Specific Namespace

```
kubectl apply -f pod.yaml -n testing
```

---

## Define Namespace Inside YAML File

To always create a resource in a specific namespace, define it in the YAML file:

```
metadata:
  name: app-pod
  namespace: testing
```

This ensures the resource is created in the correct namespace even if the command does not include `-n`.

---

# Creating a Namespace

## Using YAML

```
apiVersion: v1
kind: Namespace
metadata:
  name: testing
```

Create it using:

```
kubectl apply -f namespace.yaml
```

---

## Using Command Line

```
kubectl create namespace testing
```

---

# Switching Default Namespace

If you frequently work in a specific namespace, you can switch your current context:

```
kubectl config set-context --current --namespace=testing
```

Now, running:

```
kubectl get pods
```

Will show Pods in the `testing` namespace.

To access other namespaces, you must specify them explicitly.

---

# Viewing Resources Across All Namespaces

```
kubectl get pods --all-namespaces
```

This displays Pods from every namespace in the cluster.

---

# Resource Quotas in a Namespace

Namespaces can have resource limits.

Example:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: testing-quota
  namespace: testing
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
```

This ensures that the testing namespace:

- Cannot create more than 10 Pods
- Cannot exceed defined CPU and memory limits

---

# Summary

- A namespace divides a Kubernetes cluster into logical environments.
- Kubernetes creates default, kube-system, and kube-public automatically.
- Namespaces help isolate dev, testing, and production environments.
- Resources in the same namespace use simple names.
- Cross-namespace communication requires full DNS format.
- You can switch namespaces using kubectl config.
- Resource quotas help control usage within each namespace.

Namespaces are essential for managing large and production-grade Kubernetes clusters safely and efficiently.
