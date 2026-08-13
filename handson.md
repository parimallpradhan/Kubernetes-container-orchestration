
**Pod → Service → ReplicaSet → Deployment → Scaling → Rolling Update → Rollback → Labels/Selectors → ConfigMap → Secret → Ingress**

## 1. Start Killercoda

Open **Killercoda Kubernetes Playground** and start a Kubernetes scenario.

Once the terminal is available, first verify:

```bash
kubectl version
```

```bash
kubectl get nodes
```

```bash
kubectl get pods
```

Explain:

* `kubectl` → Kubernetes command-line tool
* `get nodes` → shows Kubernetes worker nodes
* `get pods` → shows running pods

---

# Lab 1 — Create Your First Pod

### Step 1: Create Pod YAML

```bash
vi pod.yml
```

Put:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Step 2: Create the Pod

```bash
kubectl apply -f pod.yml
```

Expected:

```text
pod/nginx-pod created
```

### Step 3: Check Pod

```bash
kubectl get pods
```

More detailed:

```bash
kubectl get pods -o wide
```

### Step 4: Describe Pod

```bash
kubectl describe pod nginx-pod
```

Tell students:

> `kubectl describe` is extremely useful when troubleshooting a Pod.

### Step 5: Check Logs

```bash
kubectl logs nginx-pod
```

### Step 6: Delete Pod

```bash
kubectl delete pod nginx-pod
```

Then:

```bash
kubectl get pods
```

Explain the important point:

> A Pod created directly does **not** automatically recreate itself when deleted.

That naturally leads to **ReplicaSet**.

---

# Lab 2 — Pod + Service

Now create the Pod again:

```bash
kubectl apply -f pod.yml
```

Check:

```bash
kubectl get pods
```

Now create a Service.

```bash
vi service.yml
```

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

But notice something important:

Our Pod currently doesn't have:

```yaml
labels:
  app: nginx
```

So modify `pod.yml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f pod.yml
```

Then:

```bash
kubectl apply -f service.yml
```

Check:

```bash
kubectl get pods
```

```bash
kubectl get svc
```

You should see something similar to:

```text
NAME            TYPE       CLUSTER-IP      PORT(S)
nginx-service   NodePort   10.x.x.x        80:30xxx/TCP
```

### Explain this carefully

```text
Service Port
     80
      |
      ↓
Target Port
     80
      |
      ↓
Pod
 nginx :80
```

Here:

* `port: 80` = Service port
* `targetPort: 80` = container application port
* `NodePort` = exposes Service through a port on the node

And yes, **Service port and targetPort can be different**.

Example:

```yaml
ports:
  - port: 8080
    targetPort: 80
```

Traffic becomes:

```text
Client
  |
  ↓
Service :8080
  |
  ↓
Pod :80
```

---

# Lab 3 — Test the Service

First:

```bash
kubectl get svc nginx-service
```

Find the NodePort.

For example:

```text
80:30080/TCP
```

Then on the Killercoda node:

```bash
curl localhost:30080
```

You should get the NGINX HTML response.

You can also test internally:

```bash
kubectl run test-pod --image=curlimages/curl -it --rm -- sh
```

Inside the container:

```bash
curl nginx-service
```

This demonstrates Kubernetes DNS:

```text
nginx-service
     |
     ↓
Kubernetes Service
     |
     ↓
nginx-pod
```

---

# Lab 4 — ReplicaSet

Now introduce the problem:

> What happens if our Pod crashes?

A ReplicaSet maintains the required number of Pods.

Create:

```bash
vi replicaset.yml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

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

Create it:

```bash
kubectl apply -f replicaset.yml
```

Check:

```bash
kubectl get pods
```

You should see 3 Pods.

```bash
kubectl get rs
```

Now delete one:

```bash
kubectl delete pod <pod-name>
```

Immediately run:

```bash
kubectl get pods
```

A replacement Pod will be created.



---

# Lab 5 — Deployment

Now explain:

> In real-world Kubernetes, we generally create Deployments rather than creating ReplicaSets manually.

Create:

```bash
vi deployment.yml
```

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
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yml
```

Check:

```bash
kubectl get deployment
```

```bash
kubectl get rs
```

```bash
kubectl get pods
```

Architecture:

```text
Deployment
     |
     ↓
ReplicaSet
     |
     +---------+---------+
     ↓         ↓         ↓
   Pod       Pod       Pod
 nginx      nginx     nginx
```

---

# Lab 6 — Scale Deployment

Current:

```yaml
replicas: 3
```

Scale to 5:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get pods
```

You should see 5 Pods.

Scale down:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

---

# Lab 7 — Rolling Update

This is an important Kubernetes demonstration.

Current image:

```yaml
image: nginx:1.27
```

Change it to:

```yaml
image: nginx:1.28
```

Then:

```bash
kubectl apply -f deployment.yml
```

Watch the rollout:

```bash
kubectl rollout status deployment nginx-deployment
```

Check:

```bash
kubectl get pods
```

Check deployment history:

```bash
kubectl rollout history deployment nginx-deployment
```

Explain:

```text
Old Pods
   ↓
nginx:1.27

       ↓ Rolling Update

New Pods
   ↓
nginx:1.28
```

Kubernetes doesn't normally kill all old Pods simultaneously during a normal Deployment rollout.

---

# Lab 8 — Rollback

Suppose version `1.28` has a problem.

Rollback:

```bash
kubectl rollout undo deployment nginx-deployment
```

Check:

```bash
kubectl rollout status deployment nginx-deployment
```

Then:

```bash
kubectl get pods
```

This is where students understand:

**Deployment = rollout + rollback + scaling + ReplicaSet management**

---

# Lab 9 — Service + Deployment

Now connect the previous concepts.

Create:

```bash
vi nginx-service.yml
```

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

Apply:

```bash
kubectl apply -f nginx-service.yml
```

Check:

```bash
kubectl get svc
```

Architecture:

```text
                  Service
              nginx-service
                   |
          Selector: app=nginx
                   |
       +-----------+-----------+
       |           |           |
       ↓           ↓           ↓
     Pod         Pod         Pod
   nginx:1.28  nginx:1.28  nginx:1.28
       ↑           ↑           ↑
       +-----------+-----------+
                   |
              ReplicaSet
                   |
              Deployment
```

This is the point where your students should understand **why labels and selectors are extremely important**.

---

# Lab 10 — ConfigMap

Create:

```bash
vi configmap.yml
```

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_ENV: "development"
  APP_NAME: "my-kubernetes-app"
```

Apply:

```bash
kubectl apply -f configmap.yml
```

Check:

```bash
kubectl get configmap
```

```bash
kubectl describe configmap app-config
```

---

# Lab 11 — Secret

Create:

```bash
vi secret.yml
```

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

type: Opaque

stringData:
  username: admin
  password: password123
```

Apply:

```bash
kubectl apply -f secret.yml
```

Check:

```bash
kubectl get secrets
```

---

# Lab 12 — Namespace

Create namespace:

```bash
kubectl create namespace dev
```

Check:

```bash
kubectl get namespaces
```

Create deployment inside it:

```bash
kubectl apply -f deployment.yml -n dev
```

Check:

```bash
kubectl get pods -n dev
```

Compare:

```bash
kubectl get pods
```

and:

```bash
kubectl get pods -n dev
```

Students learn that Kubernetes resources can be separated by namespace.

---

# Recommended Killercoda 



```text
1. Kubernetes CLI
       ↓
2. Pod
       ↓
3. Pod YAML
       ↓
4. Labels
       ↓
5. Service
       ↓
6. NodePort
       ↓
7. ReplicaSet
       ↓
8. Deployment
       ↓
9. Scaling
       ↓
10. Rolling Update
       ↓
11. Rollback
       ↓
12. ConfigMap
       ↓
13. Secret
       ↓
14. Namespace
       ↓
15. Ingress
```

## One complete project



```text
             Browser
                |
                ↓
             Ingress
                |
                ↓
          Kubernetes Service
                |
        +-------+-------+
        |       |       |
        ↓       ↓       ↓
       Pod     Pod     Pod
        |       |       |
        +-------+-------+
                |
           Deployment
                |
           ReplicaSet
```



The application should have:

* **3 replicas**
* NGINX container
* Service
* NodePort initially
* ConfigMap
* Secret
* Namespace
* Rolling update
* Rollback
* Scaling from 3 → 5
* Finally expose the application using **Ingress**


