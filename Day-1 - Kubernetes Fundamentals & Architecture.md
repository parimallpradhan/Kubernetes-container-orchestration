# Day 1 — Kubernetes Fundamentals & Architecture

### 🎯 Day 1 Goal

By the end of the session, students should be able to explain:

* What is Kubernetes?
* Why do we need Kubernetes?
* What problem does it solve?
* Kubernetes vs Docker
* Kubernetes vs Docker Swarm
* Kubernetes architecture
* Control Plane vs Worker Node
* Basic Kubernetes components
* What is a Pod?
* Basic `kubectl` commands

---

# 1. Start With a Real-World Problem

First ask students:

> "Suppose you have 20 Docker containers running your application. What happens if one container crashes?"

With plain Docker:

```text
Docker Host
   |
   +--- Container 1
   +--- Container 2
   +--- Container 3
   +--- Container 4
```

If Container 3 crashes:

```text
Container 3 ❌
```

Someone needs to:

* Detect the failure
* Restart it
* Maintain the required number of containers
* Distribute traffic
* Deploy new versions
* Roll back bad versions
* Scale applications

Doing this manually becomes difficult.

### This is where Kubernetes comes in.

---

# 2. What is Kubernetes?

Simple definition for beginners:

> **Kubernetes is an open-source container orchestration platform used to deploy, manage, scale, and maintain containerized applications.**

Break it down:

```text
Container
   ↓
Kubernetes
   ↓
Deploy
Manage
Scale
Monitor
Recover
```

Don't teach the students that Kubernetes "creates containers."

Instead explain:

```text
Developer
   ↓
Container Image
   ↓
Kubernetes
   ↓
Runs and manages containers
```

---

# 3. Why Do We Need Kubernetes?

Suppose an application has:

```text
Frontend
Backend
Database
Payment Service
Notification Service
```

And you have:

```text
10 containers
```

Then production grows:

```text
10 containers
      ↓
100 containers
      ↓
500 containers
```

Managing them manually becomes difficult.

Kubernetes helps with:

### 1. Deployment

Deploy applications automatically.

### 2. Scaling

```text
2 Pods → 5 Pods → 10 Pods
```

### 3. Self-healing

If a Pod crashes:

```text
Pod ❌
  ↓
Kubernetes detects it
  ↓
New Pod ✅
```

### 4. Load balancing

Traffic can be distributed across multiple Pods.

### 5. Rolling updates

Deploy a new application version without taking the application completely offline.

### 6. Rollback

If the new version has a problem:

```text
v2 ❌
 ↓
Rollback
 ↓
v1 ✅
```

---

# 4. Kubernetes Architecture

This is the most important part of Day 1.

Explain Kubernetes as two major parts:

```text
             Kubernetes Cluster
                    |
          +---------+---------+
          |                   |
          ↓                   ↓
     Control Plane       Worker Nodes
                            |
                    +-------+-------+
                    |       |       |
                   Pod     Pod     Pod
```

### Control Plane

The **brain** of Kubernetes.

### Worker Node

Where application workloads run.

---

# 5. Control Plane Components

Teach these four components first.

```text
             CONTROL PLANE
                  |
     +------------+------------+
     |            |            |
     ↓            ↓            ↓
 API Server   Scheduler    Controller
                              Manager
                  |
                  ↓
                 etcd
```

## 5.1 kube-apiserver

This is the **entry point to Kubernetes**.

When you execute:

```bash
kubectl get pods
```

the request goes to:

```text
kubectl
   ↓
kube-apiserver
```

The API Server communicates with the Kubernetes cluster.

Simple explanation:

> **API Server = front door of Kubernetes**

---

# 6. etcd

`etcd` stores Kubernetes cluster information.

For example:

```text
Pod information
Deployment information
Service information
Node information
Configuration/state
```

Think of it as:

> **etcd = Kubernetes cluster database**

Important:

```text
kubectl
   ↓
API Server
   ↓
etcd
```

Don't go too deep into etcd on Day 1.

---

# 7. Scheduler

Suppose you create a Pod:

```yaml
kind: Pod
```

Kubernetes needs to decide:

> "Which worker node should run this Pod?"

That's the Scheduler's job.

```text
New Pod
   ↓
Scheduler
   ↓
Worker Node 1
```

If Node 1 doesn't have enough resources:

```text
Node 1 ❌
Node 2 ✅
   ↓
Pod → Node 2
```

Simple definition:

> **Scheduler decides where a Pod should run.**

---

# 8. Controller Manager

Controllers continuously check whether the **actual state** matches the **desired state**.

Example:

You want:

```text
3 Pods
```

But currently:

```text
Pod 1 ✅
Pod 2 ✅
Pod 3 ❌
```

Controller notices:

```text
Desired = 3
Actual = 2
```

So Kubernetes creates another Pod.

```text
Pod 1 ✅
Pod 2 ✅
Pod 3 ❌
      ↓
Controller
      ↓
New Pod ✅
```

This is one of the most important concepts in Kubernetes:

> **Desired State vs Actual State**

---

# 9. Worker Node Components

Now move to the Worker Node.

```text
             Worker Node
                  |
        +---------+---------+
        |                   |
        ↓                   ↓
     kubelet           kube-proxy
        |
        ↓
 Container Runtime
        |
        ↓
    Containers
```

## kubelet

`kubelet` is the agent running on every worker node.

Its job is basically:

> "Make sure the Pods assigned to this node are running correctly."

```text
Control Plane
      |
      ↓
   kubelet
      |
      ↓
    Pod
```

---

# 10. Container Runtime

Kubernetes needs a container runtime to actually run containers.

Modern Kubernetes commonly uses:

```text
containerd
CRI-O
```

Don't spend too much time on runtime details on Day 1.

Explain:

```text
Kubernetes
    ↓
kubelet
    ↓
Container Runtime
    ↓
Container
```

---

# 11. kube-proxy

`kube-proxy` helps implement networking rules for Services on nodes.

For a fresher:

> **kube-proxy helps Kubernetes route network traffic to the appropriate Pods.**

You can explain it in more detail when teaching **Services**.

---

# 12. Complete Architecture

Now draw this for students:

```text
                         Kubernetes Cluster
                                |
             +------------------+------------------+
             |                                     |
             ↓                                     ↓
       CONTROL PLANE                          WORKER NODE
             |                                     |
     +-------+-------+                       +-----+------+
     |       |       |                       |            |
     ↓       ↓       ↓                       ↓            ↓
 API     Scheduler Controller             kubelet    kube-proxy
Server            Manager                    |
     |                                      ↓
     +-----> etcd                    Container Runtime
                                                |
                                                ↓
                                           +----+----+
                                           |         |
                                           ↓         ↓
                                         Pod 1     Pod 2
                                           |         |
                                        Container Container
```

This architecture is enough for a **Day 1 fresher**.

---

# 13. Where Does Docker Fit?

Students often ask:

> "Is Kubernetes a replacement for Docker?"

Explain:

```text
Docker
   ↓
Build container image
   ↓
Dockerfile
   ↓
Docker Image
```

Then:

```text
Kubernetes
   ↓
Deploy/manage containers
   ↓
Pods
```

Example:

```text
Developer
    |
    ↓
Dockerfile
    |
    ↓
Docker Build
    |
    ↓
Docker Image
    |
    ↓
Container Registry
    |
    ↓
Kubernetes
    |
    ↓
Pod
    |
    ↓
Container
```

This connects nicely with what students have already learned about Docker.

---

# 14. What is a Kubernetes Cluster?

A cluster is a collection of machines/nodes managed by Kubernetes.

```text
Kubernetes Cluster
       |
       +--- Control Plane
       |
       +--- Worker Node 1
       |
       +--- Worker Node 2
       |
       +--- Worker Node 3
```

Applications normally run on the Worker Nodes.

---

# 15. What is a Pod?

Don't go deeply into Pods on Day 1. Just introduce the concept.

> **Pod is the smallest deployable unit in Kubernetes.**

Basic structure:

```text
Pod
 |
 +--- Container
```

A Pod can contain one or more containers:

```text
Pod
 |
 +--- Container 1
 |
 +--- Container 2
```

For beginners, tell them:

> "For most of our initial examples, we will run one container inside one Pod."

You'll cover Pods practically on **Day 2**.

---

# 16. First Kubernetes Hands-On

Since you're using **Killercoda**, this is perfect for students.

Start a Kubernetes playground.

Then run:

```bash
kubectl version --client
```

This tells you the `kubectl` client version.

Then:

```bash
kubectl cluster-info
```

This checks the Kubernetes cluster information.

Then:

```bash
kubectl get nodes
```

Students should see something similar to:

```text
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   ...   v1.xx.x
node01         Ready    <none>          ...   v1.xx.x
```

Explain:

```text
NAME       → Node name
STATUS     → Ready/NotReady
ROLES      → Control Plane/Worker
AGE        → How long node exists
VERSION    → Kubernetes version
```

---

# 17. Important kubectl Commands for Day 1

Don't overload students.

Teach only these:

```bash
kubectl version --client
```

```bash
kubectl cluster-info
```

```bash
kubectl get nodes
```

```bash
kubectl get pods
```

```bash
kubectl get namespaces
```

```bash
kubectl get all
```

And:

```bash
kubectl describe node <node-name>
```

Explain the pattern:

```text
kubectl
   |
   +--- get
   +--- describe
   +--- create
   +--- apply
   +--- delete
```

They will use these commands repeatedly throughout the course.

---

# 18. Day 1 Mini Exercise

Give students this exercise at the end:

### Task

**1. Check Kubernetes version**

```bash
kubectl version --client
```

**2. Check cluster**

```bash
kubectl cluster-info
```

**3. List nodes**

```bash
kubectl get nodes
```

**4. Get detailed information about a node**

```bash
kubectl describe node <node-name>
```

**5. List namespaces**

```bash
kubectl get namespaces
```

**6. List Pods in all namespaces**

```bash
kubectl get pods -A
```

---

# Day 1 Interview Questions

Make sure students can answer these before moving to Day 2.

### Beginner

**Q1. What is Kubernetes?**

Container orchestration platform used to deploy, manage, scale and maintain containerized applications.

**Q2. What is a Kubernetes cluster?**

A collection of control-plane and worker-node machines managed by Kubernetes.

**Q3. What is a Control Plane?**

The components responsible for managing and controlling the Kubernetes cluster.

**Q4. What is a Worker Node?**

A machine where application workloads/Pods run.

**Q5. What is kube-apiserver?**

The API entry point through which Kubernetes components and clients communicate with the cluster.

**Q6. What is etcd?**

The distributed key-value store containing Kubernetes cluster state/configuration.

**Q7. What does Scheduler do?**

Selects an appropriate worker node for a newly created Pod.

**Q8. What does kubelet do?**

Ensures Pods assigned to its node are running according to the desired configuration.

**Q9. What is a Pod?**

The smallest deployable unit in Kubernetes.

**Q10. What is kubectl?**

The command-line tool used to interact with a Kubernetes cluster.

---

## 🎯 What students should remember from Day 1

Give them this one diagram to remember:

```text
                    KUBERNETES
                        |
                 +------+------+
                 |             |
                 ↓             ↓
           CONTROL PLANE    WORKER NODE
                 |             |
       +---------+---------+   |
       |         |         |   |
       ↓         ↓         ↓   ↓
   API Server Scheduler Controller
       |                   Manager
       ↓
      etcd                 kubelet
                              |
                              ↓
                        Container Runtime
                              |
                              ↓
                             Pod
                              |
                              ↓
                          Container
```

