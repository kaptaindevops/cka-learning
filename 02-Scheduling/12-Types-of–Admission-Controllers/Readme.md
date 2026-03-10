# Types of Admission Controllers in Kubernetes

In the previous section, we discussed **Admission Controllers** and how they intercept requests after authentication and authorization but before the object is stored in etcd.

Admission Controllers help enforce additional policies in a Kubernetes cluster.

There are **two main types of admission controllers**:

1. **Validating Admission Controllers**
2. **Mutating Admission Controllers**

Understanding the difference between these two types is important when designing cluster security and policy enforcement.

---

# Validating Admission Controllers

A **Validating Admission Controller** checks whether a request is valid and decides whether to allow or reject it.

It does **not modify the request**.

Instead, it inspects the request and returns either:

- **Allowed** → request continues
- **Denied** → request is rejected

### Example: Namespace Lifecycle Validation

Suppose a user attempts to create a Pod in a namespace that does not exist.

Example command:

```bash
kubectl run app --image=nginx -n blue
```

If the namespace `blue` does not exist, the **NamespaceLifecycle admission controller** will reject the request.

Flow of the request:

1. Request reaches API Server
2. Authentication verifies the user
3. Authorization checks permissions
4. Validating admission controller checks if namespace exists
5. Request is rejected if namespace does not exist

This type of controller simply **validates the request without modifying it**.

---

# Mutating Admission Controllers

A **Mutating Admission Controller** can modify the request before it is stored.

It may:

- Add missing fields
- Apply default values
- Modify object configuration

### Example: Default Storage Class

Consider a request to create a **PersistentVolumeClaim (PVC)**.

If the PVC does not specify a storage class, the **DefaultStorageClass admission controller** automatically adds the default storage class configured in the cluster.

Example PVC definition without storage class:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-volume
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

When the PVC is created, the admission controller automatically adds the default storage class.

If you inspect the PVC afterward, you will see that the storage class has been inserted.

This behavior demonstrates how **mutating admission controllers modify requests automatically**.

---

# Order of Execution

Admission controllers run in a specific order.

Generally:

1. **Mutating admission controllers run first**
2. **Validating admission controllers run after**

This order is important because validating controllers must inspect the **final version of the request** after mutations have been applied.

For example:

1. A mutating controller modifies a request
2. The updated request is passed to validating controllers
3. The validating controller determines whether the final request is acceptable

If a validating controller rejects the request at any point, the operation fails.

---

# Built-in Admission Controllers

Kubernetes includes several admission controllers as part of its core system.

Examples include:

- DefaultStorageClass
- LimitRanger
- NamespaceLifecycle
- ResourceQuota
- AlwaysPullImages

These controllers are compiled directly into Kubernetes and enabled through API server configuration.

---

# Extending Admission Control with Webhooks

Sometimes built-in admission controllers are not sufficient.

Organizations may need to enforce custom rules such as:

- Only allow container images from an internal registry
- Prevent usage of `latest` image tags
- Ensure Pods include mandatory labels
- Prevent containers from running as root

For these advanced policies, Kubernetes supports **Admission Webhooks**.

Admission Webhooks allow external services to validate or mutate requests.

There are two webhook types:

1. **Mutating Admission Webhook**
2. **Validating Admission Webhook**

---

# How Admission Webhooks Work

When configured, Kubernetes sends requests to an external service during the admission process.

The process works as follows:

1. User sends a request to create or modify an object
2. Request passes authentication and authorization
3. Built-in admission controllers process the request
4. Kubernetes sends the request to the webhook service
5. The webhook server evaluates the request
6. The webhook server responds with:

   - **allowed: true** → request continues  
   - **allowed: false** → request is rejected

Communication between the API server and the webhook server occurs through a JSON object called **AdmissionReview**.

This object includes details such as:

- User identity
- Operation type
- Resource type
- Object configuration

---

# Deploying an Admission Webhook

To implement a webhook-based admission controller, two main steps are required.

## 1. Deploy the Webhook Server

First, create a server that contains the logic for validation or mutation.

This server can be:

- A standalone service
- A containerized application running inside Kubernetes

It must be able to:

- Receive AdmissionReview requests
- Return responses in the expected JSON format

The server can be written in any programming language such as Go, Python, or Java.

---

## 2. Configure Kubernetes to Call the Webhook

After deploying the webhook server, Kubernetes must be configured to call it.

This is done by creating either:

- **ValidatingWebhookConfiguration**
- **MutatingWebhookConfiguration**

Example configuration structure:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: example-webhook
webhooks:
- name: validate.example.com
```

---

# Webhook Configuration Components

A webhook configuration typically includes:

### Webhook Name

A unique identifier for the webhook.

Example:

```
validate.example.com
```

---

### Client Configuration

Defines where the webhook service is located.

Two options exist:

1. **External URL**

```yaml
clientConfig:
  url: https://webhook.example.com/validate
```

2. **Kubernetes Service**

```yaml
clientConfig:
  service:
    name: webhook-service
    namespace: default
```

---

### TLS Configuration

All communication between the API server and webhook must use **TLS**.

A certificate bundle is provided using the `caBundle` field.

---

### Rules

Rules specify **when the webhook should be triggered**.

Example:

```yaml
rules:
- apiGroups: [""]
  apiVersions: ["v1"]
  operations: ["CREATE"]
  resources: ["pods"]
```

This example triggers the webhook **only when Pods are created**.

---

# Example Webhook Flow

When a Pod creation request occurs:

1. API server processes authentication and authorization
2. Built-in admission controllers run
3. API server sends request to webhook
4. Webhook server evaluates the request
5. Webhook returns response
6. API server either allows or rejects the request

---

# Summary

Admission controllers enhance Kubernetes security and policy enforcement.

Two main types exist:

- **Validating controllers** → check requests and allow or deny them
- **Mutating controllers** → modify requests before creation

Mutating controllers typically run before validating controllers.

When built-in admission controllers are not enough, Kubernetes supports **external admission webhooks**, allowing administrators to implement custom validation and mutation logic.

These webhooks provide a powerful mechanism for enforcing advanced cluster policies and security rules.
