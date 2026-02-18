# Pods in Kubernetes

## What is a Pod?

Kubernetes does not deploy containers directly on worker nodes.

Instead, containers are encapsulated inside a Kubernetes object called a **Pod**.

A Pod:

- Is a single instance of an application.
- Is the smallest object you can create in Kubernetes.
- Wraps one or more containers.

In most cases, one Pod runs one container.

---

## Basic Example

Imagine:

- A single-node Kubernetes cluster
- One application
- Running inside one Docker container
- That container is inside one Pod

So the structure looks like:

Node → Pod → Container → Application

---

## Scaling an Application

### What if traffic increases?

If more users start accessing your application, you need to scale.

Do we add more containers inside the same Pod?

No.

To scale in Kubernetes:

- You create new Pods.
- Each Pod contains a new instance of the same application.

Now you may have:

- Pod 1 → App instance 1  
- Pod 2 → App instance 2  

Both running on the same node (if capacity allows).

---

### What if the node has no capacity?

If the current node cannot handle more Pods:

- Add a new node to the cluster.
- Deploy new Pods on the new node.

This increases the physical capacity of the cluster.

---

## Pod and Container Relationship

Usually:

- One Pod = One Container

To scale:
- Scale up → Create more Pods
- Scale down → Delete existing Pods

You do NOT scale by adding more containers to an existing Pod.

---

## Can a Pod Have Multiple Containers?

Yes.

A single Pod can have multiple containers.

However:

- It is usually not multiple containers of the same type.
- Multi-container Pods are a relatively rare use case.
- Most real-world applications use one container per Pod.

---

## Why Use Multiple Containers in a Pod?

Suppose your application becomes more complex.

Example:

- Main application container
- Helper container (for logging, data processing, side tasks)

These containers:

- Need to communicate with each other
- Must share data
- Should start and stop together

Without Kubernetes, you would need to:

- Manually create Docker networks
- Configure container links
- Manage shared volumes
- Track which helper belongs to which app
- Manually stop helper containers if the main app dies

This becomes complex.

---

## How Kubernetes Helps

In Kubernetes:

- You define multiple containers inside a single Pod.
- Containers in a Pod:
  - Share the same network namespace
  - Can communicate via `localhost`
  - Share storage volumes
  - Start together
  - Stop together
  - Have the same lifecycle

This is often called the **“same fate” principle**.

Even if you start with a single container, Kubernetes forces you to use Pods.  
This makes future scaling and architectural changes easier.

---

## Deploying a Pod

You can create a Pod using:

```bash
kubectl run nginx --image=nginx
```

What happens here?

1. Kubernetes creates a Pod.
2. It pulls the specified image.
3. It runs the container inside the Pod.

---

## Where Does the Image Come From?

You must specify the image using the `--image` parameter.

Example:

```bash
kubectl run nginx --image=nginx
```

- The `nginx` image is pulled from Docker Hub by default.
- Docker Hub is a public image repository.
- You can also configure Kubernetes to pull from a private registry.

---

## Viewing Pods in the Cluster

To see the list of Pods:

```bash
kubectl get pods
```

This command shows:

- Pod name
- Status
- Number of restarts
- Age

---

## Summary

- Kubernetes does not deploy containers directly — it uses Pods.
- A Pod is the smallest deployable unit.
- Usually one Pod runs one container.
- To scale, create more Pods.
- Multi-container Pods are possible but uncommon.
- Containers inside a Pod share network and storage.
- Pods are created using `kubectl` or YAML manifests.
- Use `kubectl get pods` to list Pods.

---
