# Kubelet in Kubernetes

## Kubelet – The Captain of the Ship

The **Kubelet** can be compared to the captain of a ship.

Just like a captain manages all activities on a ship, the Kubelet manages everything happening on a Kubernetes worker node.

It is responsible for:

- Handling all necessary steps for the node to become part of the cluster.
- Acting as the single point of contact between the worker node and the control plane.
- Loading and unloading containers as instructed.
- Sending regular status reports back to the control plane.

---

## What Does the Kubelet Do?

The Kubelet runs on every worker node and performs the following tasks:

### 1. Node Registration
- Registers the worker node with the Kubernetes cluster.
- Communicates with the kube-apiserver.

### 2. Pod and Container Management
When instructed to run a Pod:
- It receives the instruction from the API Server.
- It requests the container runtime (such as Docker or containerd).
- The container runtime pulls the required image.
- It starts the container instance.

### 3. Continuous Monitoring
- Monitors the health and status of Pods and containers.
- Ensures they are running as expected.
- Reports node and Pod status back to the API Server at regular intervals.

---

## How Is the Kubelet Installed?

This is an important point.

If you use `kubeadm` to set up your cluster:

- The control plane components are deployed automatically.
- However, the Kubelet is **NOT automatically deployed on worker nodes**.
- You must manually install the Kubelet on each worker node.

---

## High-Level Installation Steps

1. Download the Kubelet binary.
2. Install and configure it.
3. Run it as a system service.
4. Verify it is running.

To verify the Kubelet process:

```bash
ps -aux | grep kubelet
```

This command helps you:
- View the running Kubelet process.
- Check the configuration options used to start it.

---

## Advanced Topics (Covered Later)

- Kubelet configuration
- Certificate generation
- TLS Bootstrapping of Kubelets

---

## Summary

- The Kubelet runs on worker nodes.
- It registers the node with the cluster.
- It manages Pods and containers using the container runtime.
- It continuously reports node and Pod status to the API Server.
- It must be manually installed on worker nodes.
