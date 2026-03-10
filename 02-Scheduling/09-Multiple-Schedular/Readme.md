# Multiple Schedulers in Kubernetes

In previous topics we learned how the **default Kubernetes scheduler** works.  
The scheduler is responsible for deciding **which node a Pod should run on**.

It evaluates several factors before placing a Pod, such as:

- Available CPU and memory on nodes
- Taints and tolerations
- Node affinity and anti-affinity rules
- Resource requests and limits
- Other scheduling constraints

This built-in scheduler works well for most use cases.

However, in some situations you may need **custom scheduling logic**.

---

# Why Use a Custom Scheduler

Sometimes an application may require special placement rules that the default scheduler cannot handle.

For example:

- Certain workloads must run only after checking external system conditions.
- Pods should run on nodes that meet custom business logic.
- Workloads must be scheduled based on organization-specific policies.

In such cases, Kubernetes allows you to **create your own scheduler**.

You can:

- Replace the default scheduler  
or
- Run **multiple schedulers in the same cluster**

This flexibility allows different applications to use different scheduling strategies.

---

# Multiple Schedulers in a Cluster

A Kubernetes cluster can run more than one scheduler at the same time.

For example:

| Scheduler | Purpose |
|---|---|
| default-scheduler | Handles most workloads |
| analytics-scheduler | Handles data analytics jobs |
| batch-scheduler | Handles batch processing |

Each scheduler must have a **unique name** so Kubernetes can identify them.

Pods can then choose which scheduler should handle them.

---

# Scheduler Configuration

Schedulers are configured using a configuration file.

Example configuration:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: analytics-scheduler
```

The important field here is:

```
schedulerName
```

This defines the name of the scheduler.

The default scheduler normally uses the name:

```
default-scheduler
```

---

# Running an Additional Scheduler

To run a new scheduler, you can start another scheduler process using:

- The same **kube-scheduler binary**
- A **custom-built scheduler binary**

The scheduler must connect to the Kubernetes API Server using a **kubeconfig file**.

Example command:

```
kube-scheduler \
--config=/etc/kubernetes/analytics-scheduler-config.yaml \
--kubeconfig=/etc/kubernetes/scheduler.conf
```

Each scheduler runs with its own configuration file.

---

# Running a Scheduler as a Pod

In modern Kubernetes setups (especially with **kubeadm**), control plane components run as Pods.

Therefore, an additional scheduler can also run as a Pod.

Example scheduler Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: analytics-scheduler
  namespace: kube-system
spec:
  containers:
  - name: scheduler
    image: k8s.gcr.io/kube-scheduler:v1.28
    command:
    - kube-scheduler
    - --config=/etc/kubernetes/scheduler-config.yaml
```

This Pod runs the scheduler inside the cluster.

---

# Leader Election

In highly available clusters, multiple scheduler instances may run across different control plane nodes.

In such setups, only **one scheduler instance should actively perform scheduling**.

This is controlled using **leader election**.

Example configuration:

```
leaderElection:
  leaderElect: true
```

Leader election ensures:

- One scheduler becomes the **active leader**
- Other schedulers remain on standby

If the leader fails, another instance automatically takes over.

---

# Deploying Scheduler Using Deployment

Schedulers can also run as a **Deployment** instead of a single Pod.

In this approach:

- Scheduler configuration is stored in a **ConfigMap**
- The configuration is mounted into the scheduler container
- Kubernetes manages the scheduler lifecycle

Example flow:

1. Create scheduler configuration
2. Store it in a ConfigMap
3. Mount the ConfigMap into the scheduler Pod
4. Deploy scheduler using Deployment

This is the common method used in production clusters.

---

# Verifying the Scheduler

After deploying the scheduler, you can check if it is running.

Example:

```bash
kubectl get pods -n kube-system
```

You should see your scheduler Pod listed.

Example output:

```
analytics-scheduler-7d8f9c
```

---

# Using a Custom Scheduler for a Pod

Once the scheduler is running, a Pod can request that scheduler.

This is done using the `schedulerName` field.

Example Pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: analytics-job
spec:
  schedulerName: analytics-scheduler
  containers:
  - name: worker
    image: nginx
```

Here:

- Kubernetes sends the Pod to **analytics-scheduler**
- The default scheduler ignores this Pod

---

# Troubleshooting Scheduling Issues

If the scheduler is not configured correctly, Pods may remain in **Pending state**.

To investigate:

```bash
kubectl describe pod <pod-name>
```

Check the **events section** for scheduling errors.

---

# Checking Which Scheduler Scheduled the Pod

You can view scheduling events using:

```bash
kubectl get events -o wide
```

Look for entries similar to:

```
Scheduled   analytics-scheduler   Successfully assigned analytics-job to node2
```

This confirms which scheduler handled the Pod.

---

# Viewing Scheduler Logs

If scheduling problems occur, check the scheduler logs.

Example:

```bash
kubectl logs analytics-scheduler -n kube-system
```

Logs help diagnose issues such as:

- Configuration errors
- API authentication failures
- Scheduling conflicts

---

# Summary

- Kubernetes uses a **default scheduler** to place Pods on nodes.
- If the default logic is not sufficient, you can create **custom schedulers**.
- A cluster can run **multiple schedulers simultaneously**.
- Each scheduler must have a unique **schedulerName**.
- Pods specify which scheduler to use using the `schedulerName` field.
- Custom schedulers can run as services, Pods, or Deployments.
- Scheduling activity can be monitored using **events and logs**.

Custom schedulers allow organizations to implement advanced scheduling strategies tailored to their specific workloads.
