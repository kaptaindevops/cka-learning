# Imperative vs Declarative Approach in Kubernetes

In this document, we will understand two important ways of managing infrastructure and Kubernetes objects:

- Imperative approach  
- Declarative approach  

Both are important, especially from a real-world and certification exam perspective.

---

# Understanding with a Simple Example

Imagine you are ordering food.

### Imperative Style

You call the restaurant and give step-by-step instructions:

- Prepare rice.
- Add vegetables.
- Cook for 10 minutes.
- Add spices.
- Pack it properly.

Here, you are telling exactly what to do and how to do it.

This is the **imperative approach**.

---

### Declarative Style

You open a food delivery app and simply order:

"One Veg Biryani."

You do not explain how to cook it.  
You just specify the final result you want.

The restaurant handles the steps internally.

This is the **declarative approach**.

You declare the desired outcome, not the steps.

---

# Imperative vs Declarative in Infrastructure

## Imperative Infrastructure

In the imperative model, you write step-by-step instructions such as:

1. Create a virtual machine.
2. Install Nginx.
3. Configure it to run on port 8080.
4. Download application source code.
5. Start the web server.

You specify both:

- What you need
- How to achieve it

If something fails midway, you must manually check what already exists and fix it.

---

## Declarative Infrastructure

In the declarative model, you simply define the desired state:

- A virtual machine named web-server
- Nginx installed
- Port 8080 configured
- Application source path defined

The system figures out:

- What already exists
- What needs to be created
- What needs to be updated

If something changes later (for example upgrading Nginx version), you update the configuration, and the system handles the rest.

This approach is more reliable and easier to manage at scale.

---

# Imperative Approach in Kubernetes

In Kubernetes, the imperative approach involves running direct commands.

Examples:

Create a Pod:
```
kubectl run nginx --image=nginx
```

Create a Deployment:
```
kubectl create deployment web --image=nginx
```

Expose a Deployment:
```
kubectl expose deployment web --port=80
```

Scale a Deployment:
```
kubectl scale deployment web --replicas=3
```

Update an image:
```
kubectl set image deployment/web nginx=nginx:1.25
```

Edit a resource:
```
kubectl edit deployment web
```

These commands:

- Quickly create or modify resources
- Do not require YAML files
- Are useful during exams

However, they have limitations:

- Hard to track changes
- Not suitable for complex configurations
- No version control
- Difficult for team collaboration

---

# Using Configuration Files (Still Imperative)

Another method is using YAML files with commands like:

Create:
```
kubectl create -f deployment.yaml
```

Update:
```
kubectl replace -f deployment.yaml
```

Delete:
```
kubectl delete -f deployment.yaml
```

This is still considered imperative because:

- You explicitly tell Kubernetes when to create or replace.
- You must know whether the object already exists.
- If the object exists, create fails.
- If the object does not exist, replace fails.

You must manually manage the state.

---

# The Problem with kubectl edit

When you run:

```
kubectl edit deployment web
```

You modify the live object stored in the cluster.

However:

- Your local YAML file is not updated.
- The change is not recorded in version control.
- Future updates may overwrite your changes.

This can cause confusion in team environments.

A better approach:

1. Update the local YAML file.
2. Run:
   ```
   kubectl replace -f deployment.yaml
   ```

This keeps your configuration consistent and trackable.

---

# Declarative Approach in Kubernetes

The declarative method uses:

```
kubectl apply
```

Instead of create or replace.

Example:

```
kubectl apply -f deployment.yaml
```

What makes apply different?

- If the object does not exist → it creates it.
- If the object exists → it updates it.
- It compares the current state with the desired state.
- It only applies necessary changes.

You do not need to check manually.

---

# Applying Multiple Files

You can apply an entire directory:

```
kubectl apply -f ./k8s-config/
```

This creates or updates all objects inside that folder.

For future updates:

- Modify YAML files
- Run apply again

Kubernetes automatically figures out what changed.

---

# Why Declarative Is Better for Production

- Easy to track in Git
- Supports review and approval processes
- Safer for teams
- Better for large environments
- Handles updates intelligently

This is the preferred approach in real production environments.

---

# Exam Tips

For certification exams:

Use imperative commands when:

- Quickly creating simple Pods or Deployments
- Updating small properties
- Saving time

Use YAML files when:

- Requirements are complex
- Multiple containers are involved
- Environment variables are required
- Init containers are needed
- Volume configurations are required

If you make a mistake in YAML:

- Update the file
- Run:
  ```
  kubectl apply -f file.yaml
  ```

This is often safer than manually editing live objects.

---

# Summary

- Imperative approach: Specify how to do something.
- Declarative approach: Specify the final desired state.
- kubectl create, edit, scale are imperative.
- kubectl apply is declarative.
- Declarative is preferred for production.
- Imperative commands are useful during exams.

Understanding both approaches is essential for managing Kubernetes efficiently and performing well in certification exams.
