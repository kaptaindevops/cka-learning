# Kubernetes Architecture Explained Using a Company Example

Think of Kubernetes like a company.

There are two main parts:

- Management Team (Control Plane)
- Employees (Worker Nodes)

---

## Company = Kubernetes Cluster

The entire company represents the Kubernetes cluster.

---

## Management Team = Control Plane

Management does not directly do technical work.

Instead, they:

- Assign tasks
- Maintain company records
- Ensure required staff is present
- Monitor overall operations

### Components in This Example

#### kube-apiserver = CEO

- Central authority.
- All decisions pass through here.
- Communicates with all departments.

#### etcd = Company Database

- Stores employee records.
- Keeps project information.
- Maintains company configuration.

#### kube-scheduler = HR Team

- Assigns employees to projects.
- Checks workload before assigning tasks.

#### Controller Manager = Operations Team

- Ensures required number of employees are working.
- Replaces employees if someone leaves.
- Maintains operational balance.

---

## Employees = Worker Nodes

This is where actual work gets done.

#### kubelet = Employee

- Receives assigned tasks.
- Completes assigned work (runs containers).
- Reports status back to management.

#### kube-proxy = Internal Communication System

- Enables communication between employees.
- Ensures teams can collaborate.

#### Container Runtime = Employee Laptop

- Used to perform actual work.
- Without laptop, employee cannot work.

---

## Simple Flow

Client request → CEO approves → HR assigns employee → Employee completes task → Operations team ensures everything stays stable.
