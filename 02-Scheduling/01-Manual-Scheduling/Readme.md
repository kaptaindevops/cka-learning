# Scheduling Without a Scheduler in Kubernetes

What happens if your cluster does not have a scheduler running?

In a normal Kubernetes setup, the default scheduler continuously watches for newly created Pods and assigns them to suitable Nodes. But if there is no scheduler available, Pods will not be placed automatically.

Let us understand how scheduling works internally and what options you have if the scheduler is not present.

---

# How the Scheduler Works Internally

When you create a Pod, its definition does not usually include a `nodeName` field.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: api-pod
spec:
  containers:
    - name: api-container
      image: nginx
```

Notice that we do not define which Node the Pod should run on.

Here is what happens in the background:

1. The Pod is created without a `nodeName`.
2. The scheduler monitors the cluster.
3. It looks for Pods that do not have a `nodeName` assigned.
4. These Pods become candidates for scheduling.
5. The scheduler runs its algorithm to find a suitable Node.
6. Once selected, it assigns the Pod to that Node by creating a **Binding object**.
7. The `nodeName` field is set automatically.

This is how Pods move from **Pending** to **Running** state.

---

# What Happens If There Is No Scheduler?

If the scheduler is not running:

- New Pods remain in the **Pending** state.
- No Node is assigned.
- Workloads do not start.

Since no component is responsible for assigning Nodes, you must handle scheduling manually.

---

# Option 1: Manual Scheduling at Creation Time

The simplest method is to specify the `nodeName` field while creating the Pod.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: analytics-pod
spec:
  nodeName: worker-node-1
  containers:
    - name: analytics-container
      image: nginx
```

By specifying `nodeName`, you bypass the scheduler completely.

Kubernetes directly assigns the Pod to the given Node.

Important points:

- This works only at creation time.
- You must know the exact Node name.
- No scheduling logic or validation is applied.

---

# Can You Modify nodeName After Creation?

No.

Once a Pod is created, Kubernetes does not allow you to modify the `nodeName` field.

If the Pod already exists and is still Pending, you cannot simply edit the Pod and add `nodeName`.

You must use another approach.

---

# Option 2: Creating a Binding Object (Advanced Method)

The scheduler assigns Pods to Nodes using a **Binding object**.

You can mimic this behavior manually.

Steps involved:

1. Create a Binding object.
2. Specify the target Node.
3. Send a POST request to the Kubernetes API.

Example Binding structure (conceptually):

```
{
  "apiVersion": "v1",
  "kind": "Binding",
  "metadata": {
    "name": "analytics-pod"
  },
  "target": {
    "apiVersion": "v1",
    "kind": "Node",
    "name": "worker-node-1"
  }
}
```

This JSON is sent to the Pod binding API endpoint.

This method:

- Imitates what the scheduler does internally.
- Assigns an existing Pending Pod to a Node.
- Requires API-level interaction.

It is rarely used in day-to-day operations but helps understand how scheduling works internally.

---

# Key Takeaways

- The scheduler assigns Pods to Nodes by setting the `nodeName` field.
- It monitors Pods without a `nodeName`.
- If no scheduler is running, Pods remain in Pending state.
- You can manually schedule a Pod by specifying `nodeName` during creation.
- You cannot modify `nodeName` after creation.
- Advanced scheduling can be done using a Binding object.

Understanding this flow gives you deeper insight into how Kubernetes makes scheduling decisions behind the scenes.
