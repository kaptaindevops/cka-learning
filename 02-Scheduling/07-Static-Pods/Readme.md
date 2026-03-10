# Static Pods in Kubernetes

Earlier in the course we learned about the Kubernetes architecture and how different components work together.  
The **Kubelet** is responsible for running containers on a node, but normally it receives instructions from the **Kubernetes control plane**.

In a typical cluster:

1. The **API Server** receives requests.
2. The **Scheduler** decides which node should run the Pod.
3. The decision is stored in **etcd**.
4. The **Kubelet** reads the instruction from the API Server and starts the Pod on the node.

But what happens if the node is **not part of a Kubernetes cluster**?

What if there is:

- No API Server
- No Scheduler
- No Controllers
- No etcd
- No control plane at all

Can the Kubelet still run Pods?

Yes. The Kubelet can operate independently using something called **Static Pods**.

---

# Running Pods Without a Kubernetes Cluster

Even without a control plane, a node can still run containers if the following are available:

- Kubelet
- A container runtime (such as containerd)

However, since there is no API Server, the Kubelet has no source from which it can receive Pod instructions.

To solve this, the Kubelet can be configured to read Pod definitions directly from a **local directory**.

---

# How Static Pods Work

The Kubelet periodically checks a specific directory on the node.

If it finds a Pod definition file in that directory:

- It reads the file
- Creates the Pod
- Ensures the Pod keeps running

This directory is known as the **Static Pod Manifest Path**.

The behavior is automatic:

- If a new Pod file is added → the Pod is created
- If the file is modified → the Pod is recreated
- If the file is removed → the Pod is deleted

These Pods created directly by the Kubelet are called **Static Pods**.

---

# Important Limitation

Static Pods support **only Pod objects**.

You cannot create the following using static pod files:

- Deployments
- ReplicaSets
- Services
- StatefulSets

These objects require Kubernetes control plane components.

Since the Kubelet only understands Pods, static pods are limited to Pod definitions.

---

# Configuring the Static Pod Directory

The Kubelet needs to know where the Pod manifest files are located.

This is configured using the **Pod Manifest Path** option.

Example configuration:

```
--pod-manifest-path=/etc/kubernetes/manifests
```

This means the Kubelet will continuously monitor the directory:

```
/etc/kubernetes/manifests
```

Any Pod definition placed there will be automatically started.

---

# Alternative Configuration Method

Instead of specifying the path directly in the Kubelet service, the configuration can be provided through a configuration file.

Example:

```
staticPodPath: /etc/kubernetes/manifests
```

This method is commonly used when the cluster is created using **kubeadm**.

---

# How to Find the Static Pod Directory

If you are inspecting an existing cluster and want to find the static pod directory:

1. Check the Kubelet service configuration for the option:

```
--pod-manifest-path
```

2. If it is not present, check the configuration file specified using:

```
--config
```

3. Inside that configuration file, look for:

```
staticPodPath
```

This will reveal the directory where static Pod manifests are stored.

---

# Viewing Static Pods Without a Cluster

If the node is not part of a cluster, there is no API Server.

Because of this:

- The `kubectl` command cannot be used.
- Pod information must be retrieved from the container runtime.

Example:

```
docker ps
```

or

```
crictl ps
```

These commands show running containers managed by the Kubelet.

---

# Static Pods in a Kubernetes Cluster

When the node is part of a cluster, the Kubelet can receive instructions from two sources:

1. Static Pod manifest files
2. The Kubernetes API Server

This means the Kubelet can run:

- Static Pods
- Regular Pods scheduled by the control plane

Both can run on the same node simultaneously.

---

# Mirror Pods

When a static Pod is created on a node that is part of a cluster, the Kubelet also creates a **mirror object** in the API Server.

This mirror Pod:

- Appears in `kubectl get pods`
- Is visible in the cluster
- Is **read-only**

You cannot edit or delete the mirror Pod using kubectl.

To remove the Pod, you must delete the manifest file from the node.

---

# Naming of Static Pods

Static Pods appear in the cluster with the node name appended.

Example:

```
kube-apiserver-node01
```

This indicates that the Pod is running as a static Pod on `node01`.

---

# Why Static Pods Are Useful

Static Pods are commonly used to run **Kubernetes control plane components**.

Examples:

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd

In kubeadm-based clusters, these components run as static Pods.

The Kubelet manages them directly.

Benefits include:

- Automatic restart if a component crashes
- Simpler configuration
- No need to manually manage system services

---

# Static Pods vs DaemonSets

Static Pods and DaemonSets may appear similar, but they work differently.

### Static Pods

- Created directly by the Kubelet
- Do not depend on the API Server
- Defined using local manifest files
- Typically used for control plane components

### DaemonSets

- Managed by the Kubernetes control plane
- Created using the API Server
- Ensure one Pod runs on every node
- Commonly used for logging agents or monitoring tools

Both types of Pods are **not scheduled by the Kubernetes Scheduler**.

---

# Summary

- The Kubelet can run Pods even without a Kubernetes cluster.
- Static Pods are created from local Pod manifest files.
- The Kubelet continuously monitors the manifest directory.
- Static Pods are commonly used to run Kubernetes control plane components.
- When part of a cluster, static Pods appear as mirror Pods in the API Server.
- Static Pods are different from DaemonSets and are managed directly by the Kubelet.
