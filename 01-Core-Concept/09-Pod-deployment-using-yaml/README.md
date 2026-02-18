# Understanding `pod-definition.yaml` in Kubernetes

In this document, we will break down a basic Pod YAML file and understand each section clearly in a simple and practical way.

---

## YAML File Location

Instead of pasting the full YAML content here, you can view the actual file from the GitHub repository:

[pod-definition.yaml](./pod-definition.yaml)

This file contains the basic Pod definition used to deploy an Nginx container.

---

# Explanation of Each Section

Below is a detailed explanation of the structure inside the `pod-definition.yaml` file.

---

# 1. apiVersion

- Defines which Kubernetes API version is used.
- `v1` is used for core resources such as:
  - Pod
  - Service
  - ConfigMap
  - Secret

This tells Kubernetes which API version should interpret the object.

---

# 2. kind

- Specifies the type of Kubernetes object.
- In this case, it is a **Pod**.

Other possible values for `kind` include:
- Deployment
- Service
- ReplicaSet
- ConfigMap

Here, we are directly creating a Pod.

---

# 3. metadata

The `metadata` section contains identification details of the Pod.

It typically includes:

## name

- The name of the Pod.
- Must be unique within a namespace.
- Used for:
  - Viewing the Pod
  - Deleting the Pod
  - Describing the Pod
  - Checking logs

Example:

```bash
kubectl get pod <pod-name>
```

---

## labels

- Labels are key-value pairs.
- Used to organize and group resources.
- Services and Deployments use labels to select Pods.

Example:

```bash
kubectl get pods -l app=nginx
```

Labels are very important for networking and scaling.

---

# 4. spec

The `spec` section defines the desired state of the Pod.

It describes:

- Which containers should run
- Which images should be used
- Which ports should be exposed
- Runtime configuration details

Everything related to the container execution is defined inside `spec`.

---

# 5. containers

The `containers` section defines one or more containers inside the Pod.

Most commonly:

- One Pod runs one container.
- Multi-container Pods are possible but less common.

Each container definition includes:

## container name

- Unique name of the container inside the Pod.
- Useful for debugging and fetching logs.

Example:

```bash
kubectl logs <pod-name> -c <container-name>
```

---

## image

- Specifies the container image to run.
- By default, images are pulled from Docker Hub.
- Format: `image-name:tag`
- If no tag is specified, `latest` is assumed.

For production, always use a specific version instead of `latest`.

---

## containerPort

- Defines the port exposed by the container.
- Helps when creating Services.
- Does not automatically expose the Pod externally.

To expose it outside the cluster, you need a Service.

---

# What Happens When You Apply This File?

When you run:

```bash
kubectl apply -f pod-definition.yaml
```

Kubernetes performs the following steps:

1. The request goes to the API Server.
2. The Pod definition is stored in etcd.
3. The Scheduler selects a suitable Node.
4. The Kubelet on that Node:
   - Pulls the container image.
   - Creates the container.
   - Starts the Pod.

---

# Important Notes

- This creates a standalone Pod.
- If the Pod is deleted, it will not be recreated automatically.
- For production environments, use a Deployment instead of a raw Pod.
- Pods are usually managed by higher-level controllers like Deployments.

---

# Summary

- `apiVersion` defines the API version.
- `kind` defines the resource type.
- `metadata` identifies the Pod.
- `spec` defines how the Pod should run.
- `containers` define what runs inside the Pod.
- `image` defines the container source.
- `containerPort` defines the exposed port inside the container.

---
