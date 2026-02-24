# Kubernetes Services – Connecting Applications Inside and Outside the Cluster

Kubernetes Services enable communication between different parts of an application and also allow users outside the cluster to access applications.

In a real-world microservices application, you may have:

- A group of frontend Pods serving users
- A group of backend Pods processing data
- A database or external data source

Services make communication between these components possible. They also allow applications running inside the cluster to be accessed externally.

---

# Why Do We Need Services?

Pods are dynamic in nature:

- Their IP addresses change if they are recreated.
- They can be distributed across multiple nodes.
- They can scale up or down.

Because of this, directly connecting to Pod IPs is not reliable.

A Service provides:

- A stable IP address
- A stable DNS name
- Automatic load balancing across Pods
- Decoupling between application components

This allows loose coupling between microservices.

---

# External Access Problem – A Simple Scenario

Assume:

- Kubernetes Node IP: `192.168.1.2`
- Your laptop IP: `192.168.1.1`
- Pod IP: `10.244.0.2`

The Pod IP belongs to the internal cluster network.  
You cannot directly access `10.244.0.2` from your laptop.

You could SSH into the node and access the Pod internally, but that is not practical.

What we need is a way to:

- Send traffic to the node
- Have the node forward traffic to the Pod

This is exactly what a Kubernetes Service does.

---

# Types of Kubernetes Services

Kubernetes provides three main types of Services:

## 1. NodePort

- Exposes a Pod on a port of each Node.
- Allows external access using:
  
  ```
  NodeIP:NodePort
  ```

- Suitable for testing or simple external access.

---

## 2. ClusterIP (Default)

- Creates a virtual IP inside the cluster.
- Used for internal communication between services.
- Example: frontend communicating with backend.

This is the most commonly used type.

---

## 3. LoadBalancer

- Provisions an external load balancer (in supported cloud providers).
- Used in production environments.
- Distributes traffic across multiple Pods.

---

# Understanding NodePort Service

Let’s focus on NodePort.

When using NodePort, three ports are involved:

### 1. TargetPort
- The port on the Pod.
- Where the actual application is running.
- Example: 80 (Nginx default port)

### 2. Port
- The port of the Service itself.
- Internal cluster port.

### 3. NodePort
- The port exposed on each Node.
- Used to access the application externally.
- Must be between 30000 and 32767.

Example:

```
http://192.168.1.2:30008
```

---

# NodePort Range

By default:

- Valid NodePort range: 30000–32767
- If not specified, Kubernetes automatically assigns one.

---

# Structure of a Service YAML File

A Service definition file contains:

- apiVersion: v1
- kind: Service
- metadata
- spec

Example file:

[service-definition.yaml](./service-definition.yaml)

---

# Important Fields in Service Spec

## type

Defines the Service type:

- ClusterIP
- NodePort
- LoadBalancer

---

## ports

An array containing port mappings.

- `port` → Mandatory
- `targetPort` → Defaults to `port` if not specified
- `nodePort` → Optional for NodePort (auto-assigned if omitted)

---

## selector

This connects the Service to Pods.

Example:

```
selector:
  app: myapp
```

The Service will forward traffic only to Pods that have:

```
labels:
  app: myapp
```

This is how Services find and manage the correct Pods.

---

# Service with Multiple Pods

In production, you typically run multiple Pods for high availability.

If three Pods have the same label:

```
app: myapp
```

And your Service selector matches that label:

- The Service automatically detects all three Pods.
- It distributes incoming traffic among them.
- No extra configuration is required.

Kubernetes uses a random load balancing algorithm by default.

---

# Service Across Multiple Nodes

Even if Pods are running on different nodes:

- The Service automatically spans across all nodes.
- The same NodePort is opened on every node.
- You can access the application using any node’s IP with the same port.

Example:

```
Node1:30008
Node2:30008
Node3:30008
```

All will route traffic correctly to available Pods.

---

# Automatic Updates

If:

- A Pod is deleted
- A new Pod is created
- Pods scale up or down

The Service automatically updates its endpoints.

You do not need to manually reconfigure anything.

This makes Services flexible and adaptive.

---

# Summary

- Services enable communication inside and outside the cluster.
- They provide stable networking for dynamic Pods.
- NodePort exposes applications externally.
- ClusterIP enables internal communication.
- LoadBalancer integrates with cloud providers.
- Services use labels and selectors to identify Pods.
- They automatically load balance traffic.
- They adapt automatically when Pods are added or removed.

Kubernetes Services are a fundamental building block for building scalable and production-ready applications.
