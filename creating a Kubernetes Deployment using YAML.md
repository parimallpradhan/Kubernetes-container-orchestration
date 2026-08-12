# Creating a Kubernetes Deployment Using YAML

## 1. First explain: What is a Deployment?

Tell students:

> **A Deployment is a Kubernetes object used to manage and maintain a desired number of Pod replicas and support controlled application updates.**

For example, instead of manually creating 3 Pods:

```text
Pod 1
Pod 2
Pod 3
```

we tell Kubernetes:

```text
I want 3 replicas of my application.
```

Kubernetes manages them for us.

```text
              Deployment
                   |
              ReplicaSet
                   |
        +----------+----------+
        |          |          |
       Pod        Pod        Pod
        |          |          |
    Container  Container  Container
```

---

# 2. Create the Deployment YAML

Create a file:

```bash
nano deployment.yaml
```

Put this inside:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

# 3. Explain it line by line

Don't just make students copy it.

### `apiVersion`

```yaml
apiVersion: apps/v1
```

This tells Kubernetes which API version is used for the Deployment resource.

---

### `kind`

```yaml
kind: Deployment
```

This tells Kubernetes:

> "I want to create a Deployment."

---

### `metadata`

```yaml
metadata:
  name: nginx-deployment
```

This gives the Deployment its name.

You can verify it later with:

```bash
kubectl get deployments
```

---

# 4. Explain `spec`

```yaml
spec:
```

This contains the **desired configuration** of the Deployment.

Then:

```yaml
replicas: 3
```

means:

> "I want 3 Pods."

So:

```text
Desired State

3 Pods
```

---

# 5. Explain selector

This is very important:

```yaml
selector:
  matchLabels:
    app: nginx
```

It tells the Deployment which Pods it manages.

---

# 6. Explain Pod template

This section:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

defines the Pods that the Deployment will create.

Notice:

```yaml
selector:
  matchLabels:
    app: nginx
```

and:

```yaml
labels:
  app: nginx
```

match.

Show students:

```text
Deployment
    |
    | selector: app=nginx
    |
    v
Pod Template
    |
    | label: app=nginx
    |
    v
Pods
```

### Important teaching point

> **The Deployment selector must match the labels on its Pod template.**

---

# 7. Explain container configuration

Inside the Pod template:

```yaml
containers:
  - name: nginx
    image: nginx:latest
```

This tells Kubernetes:

> Create an Nginx container using the `nginx:latest` image.

And:

```yaml
ports:
  - containerPort: 80
```

documents that the container listens on port 80.

Make it clear:

> `containerPort` does **not** by itself expose the application outside the cluster. A Service is typically used for that.

That's an important beginner misconception to prevent.

---

# 8. Apply the Deployment

Run:

```bash
kubectl apply -f deployment.yaml
```

Expected:

```text
deployment.apps/nginx-deployment created
```

---

# 9. Check the Deployment

```bash
kubectl get deployments
```

You should see something similar to:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

Explain:

```text
3/3
```

means:

> 3 desired replicas and 3 ready replicas.

---

# 10. Check the ReplicaSet

Now run:

```bash
kubectl get replicasets
```

You should see something similar to:

```text
NAME                          DESIRED   CURRENT   READY
nginx-deployment-xxxxxxxxxx   3         3         3
```

Explain the relationship:

```text
Deployment
     |
     v
ReplicaSet
     |
 +---+---+---+
 |   |   |   |
Pod Pod Pod Pod
```

For 3 replicas:

```text
Deployment
     |
ReplicaSet
     |
 +---+---+
 |   |   |
Pod Pod Pod
```

---

# 11. Check the Pods

```bash
kubectl get pods
```

You should get something like:

```text
NAME                                READY   STATUS
nginx-deployment-xxxxx-abcde        1/1     Running
nginx-deployment-xxxxx-fghij        1/1     Running
nginx-deployment-xxxxx-klmno        1/1     Running
```

Point out:

> Students didn't create these Pods manually.

They only created:

```text
Deployment
```

Kubernetes created:

```text
ReplicaSet
    ↓
Pods
```

This is the power of a Deployment.

---

# 12. See where Pods are running

```bash
kubectl get pods -o wide
```

This gives additional information including the node where each Pod is running.

For example:

```text
NAME                     STATUS    IP          NODE
nginx-xxx                Running   10.244...   worker1
nginx-yyy                Running   10.244...   worker2
nginx-zzz                Running   10.244...   worker1
```

This connects nicely with the Kubernetes architecture class.

---

# 13. Describe the Deployment

Run:

```bash
kubectl describe deployment nginx-deployment
```

Show students the important sections:

```text
Replicas
Selector
Pod Template
Conditions
Events
```

Events are especially useful when troubleshooting.

---

# 14. Demonstrate Self-Healing

This is a great classroom demo.

First:

```bash
kubectl get pods
```

Suppose:

```text
Pod A
Pod B
Pod C
```

Delete one:

```bash
kubectl delete pod <pod-name>
```

Immediately check:

```bash
kubectl get pods
```

Students will see Kubernetes create a replacement Pod.

Explain:

```text
Desired = 3

Current:
Pod A
Pod B
Pod C → deleted

Current = 2

        ↓

ReplicaSet detects mismatch

        ↓

Creates replacement

        ↓

Current = 3
```

### Key concept

> **The Deployment/ReplicaSet helps maintain the desired number of Pods.**

---

# 15. Demonstrate Scaling

Initially:

```yaml
replicas: 3
```

Change it to:

```yaml
replicas: 5
```

Then:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
```

Now you should have 5 Pods.

Explain:

```text
3 Pods
  ↓
Change desired state
  ↓
5 Pods
```

You can also demonstrate scaling imperatively:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Then explain:

> "The command changes the live resource. The YAML file remains the source configuration unless you update it too."

This is an excellent opportunity to explain **configuration drift**.

---

# 16. Demonstrate Rolling Update

Change:

```yaml
image: nginx:latest
```

to a specific newer Nginx tag available in your lab, for example:

```yaml
image: nginx:<version>
```

Then:

```bash
kubectl apply -f deployment.yaml
```

Watch:

```bash
kubectl rollout status deployment/nginx-deployment
```

And:

```bash
kubectl get pods
```

Explain:

```text
Old Pods
   ↓
New Pods created gradually
   ↓
Old Pods terminated
   ↓
New version running
```

This is a **rolling update**.

---

# 17. Show Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

If you have a revision history, Kubernetes can show the rollout revisions.

---

# 18. Demonstrate Rollback

If the new version has a problem:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Then:

```bash
kubectl rollout status deployment/nginx-deployment
```

Explain:

```text
Version 1
   ↓
Version 2
   ↓
Problem
   ↓
Rollback
   ↓
Version 1
```

This connects directly to your previous **Docker Swarm vs Kubernetes** class.

---

# 19. Important: Deployment does NOT expose the application

At this point, students may try:

```text
Browser → Node IP → Nginx
```

and wonder why it doesn't work.

Explain:

```text
Deployment
    |
    v
Pods
    |
    v
Nginx
```

The Deployment manages the application Pods.

For network access, you normally create a:

```text
Service
```

Architecture:

```text
                 User
                  |
                  v
               Service
                  |
        +---------+---------+
        |         |         |
       Pod       Pod       Pod
        |         |         |
      Nginx     Nginx     Nginx
```

That's your perfect transition into the next topic:

# Deployment → Service

---

# 20. Useful commands students should practice

```bash
kubectl apply -f deployment.yaml
```

```bash
kubectl get deployment
```

```bash
kubectl get rs
```

```bash
kubectl get pods
```

```bash
kubectl get pods -o wide
```

```bash
kubectl describe deployment nginx-deployment
```

```bash
kubectl rollout status deployment/nginx-deployment
```

```bash
kubectl rollout history deployment/nginx-deployment
```

```bash
kubectl rollout undo deployment/nginx-deployment
```

```bash
kubectl delete -f deployment.yaml
```

---

# 21. The most important diagram for students

Make sure they understand this:

```text
                 deployment.yaml
                       |
                       v
                  Deployment
                       |
                       v
                  ReplicaSet
                       |
          +------------+------------+
          |            |            |
          v            v            v
        Pod 1        Pod 2        Pod 3
          |            |            |
       Nginx         Nginx         Nginx
      Container     Container     Container
```

And the control flow:

```text
YAML
  |
kubectl apply
  |
API Server
  |
Deployment
  |
ReplicaSet
  |
Pods
  |
Container Runtime
  |
Nginx Containers
```

## 🎯 Give students this practical exercise

Ask them to implement:

**Requirement:**

> Deploy an Nginx application using Kubernetes Deployment YAML with **3 replicas**. Verify the Pods and ReplicaSet. Delete one Pod and observe self-healing. Change replicas from 3 → 5. Finally, perform a rolling update and rollback.

That single exercise teaches them **YAML + Deployment + ReplicaSet + Pods + desired state + self-healing + scaling + rollout + rollback** in one lab.
