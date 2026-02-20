# Kubernetes Mastery

> **How to use this document:** Read a section. Understand the mental model. Do the hands-on lab. Then try to explain it out loud to an imaginary junior engineer. If you can't explain it, reread. This is the Feynman technique — the only way to know if you truly understand something.

---

## Before Kubernetes: Why Does It Even Exist?

You need to understand the *pain* Kubernetes was built to solve, otherwise it's just a pile of YAML files and magic commands.

### The Old World: Running Apps on Servers

Imagine you're at a startup in 2010. You have an app. You SSH into a server, run your app, and go home. Life is simple.

Then your app gets popular. You need 10 servers. Then 100. Problems start:

- **"It works on my machine"** — your app works locally but crashes in production because the server has a different Python version, different environment variables, different file paths.
- **Deployment is manual** — someone SSHes into every server, pulls new code, restarts the app. This takes hours and humans make mistakes.
- **Servers are snowflakes** — Server A has been patched 47 times, Server B hasn't. They're different. You don't know why things break.
- **No self-healing** — your app crashes at 3am, it stays crashed until someone wakes up and restarts it.
- **Resource waste** — your app uses 10% of CPU on a server that's paying for 100%. The rest is wasted.

### The Container Revolution: Docker

Before Kubernetes, there was Docker. Docker solved the "works on my machine" problem by packaging your app and everything it needs (runtime, libraries, config) into a single unit called a **container**.

Think of a container like a shipping container on a cargo ship. Before shipping containers, you'd load bananas from Ecuador, TVs from Japan, and clothing from Bangladesh all differently — each cargo type needed different handling. It was chaos. The shipping container standardized everything: same box, same cranes, same ships, regardless of what's inside.

Docker did the same for software. Your app, regardless of what language or dependencies it needs, gets put in a standard container. Any machine that can run Docker can run your container, identically.

**But Docker alone isn't enough at scale.** If you have 500 containers across 50 servers:
- How do you decide which server runs which container?
- What happens when a server dies?
- How do containers talk to each other?
- How do you update 500 containers without downtime?
- How do you scale up when traffic spikes?

This is where **Kubernetes** (K8s) comes in. Kubernetes is a **container orchestration system** — it manages containers at scale. It answers all those questions automatically.

---

## The Mental Model: Kubernetes as an Autonomous Operations Team

Here's the single most important mental model for Kubernetes:

**You tell Kubernetes what you WANT. Kubernetes figures out HOW to make it happen and keeps it that way.**

This is called **declarative configuration** vs **imperative commands**.

- **Imperative:** "SSH into server 3, start 5 copies of my app, then SSH into server 7..."
- **Declarative:** "I want 5 copies of my app running at all times." Kubernetes does the rest — and if one crashes, Kubernetes restarts it. If a server dies, Kubernetes moves those containers elsewhere.

Think of it like a thermostat vs a heater switch. A heater switch is imperative — you turn it on and off manually. A thermostat is declarative — you say "I want 22°C" and it figures out when to heat and when to stop. Kubernetes is your infrastructure thermostat.

---

## The Architecture: What's Actually Running

Before diving into individual concepts, you need to see the whole picture.

```
┌─────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                     │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              CONTROL PLANE (the brain)            │   │
│  │                                                    │   │
│  │  API Server ←→ etcd (state store)                 │   │
│  │      ↕                                             │   │
│  │  Scheduler    Controller Manager                  │   │
│  └──────────────────────────────────────────────────┘   │
│              ↕ (gives orders to workers)                 │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐           │
│  │  NODE 1   │  │  NODE 2   │  │  NODE 3   │           │
│  │           │  │           │  │           │           │
│  │ kubelet   │  │ kubelet   │  │ kubelet   │           │
│  │ kube-proxy│  │ kube-proxy│  │ kube-proxy│           │
│  │           │  │           │  │           │           │
│  │ [Pod][Pod]│  │ [Pod][Pod]│  │ [Pod]     │           │
│  └───────────┘  └───────────┘  └───────────┘           │
└─────────────────────────────────────────────────────────┘
```

### Control Plane (The Brain)

The control plane is where decisions are made. In production, this runs on dedicated machines you don't touch. In your local setup (minikube), it all runs on your laptop.

**API Server:** Every single thing in Kubernetes goes through the API Server. You run `kubectl get pods` — that's a call to the API Server. A pod crashes and Kubernetes restarts it — the controller talked to the API Server to do that. It's the front door to the entire cluster. It's stateless — all state is stored in etcd.

**etcd:** A distributed key-value database that stores the entire state of your cluster. What pods should be running? What's the current state? All of it lives here. If etcd dies and you have no backup, your cluster is gone. In production, etcd is replicated across multiple machines.

**Scheduler:** Watches for new pods that don't have a node assigned yet, and picks which node they should run on. It considers: How much CPU/memory does the pod need? How much is available on each node? Are there any special rules (only run on nodes with GPUs, etc.)?

**Controller Manager:** A collection of controllers (programs) that watch the state of the cluster and take action to move toward the desired state. The Deployment Controller watches Deployments. If you say "I want 3 replicas" and only 2 exist, the Deployment Controller creates the third one. Every K8s resource type has a controller watching it.

### Nodes (The Workers)

Nodes are the machines (VMs or physical servers) that actually run your containers. Each node runs:

**kubelet:** An agent that talks to the API Server. The Control Plane says "run this pod on node 2" — the kubelet on node 2 is what actually makes that happen. It constantly reports back: "Pod X is running. Pod Y is dead."

**kube-proxy:** Handles networking. When traffic needs to reach a Service (we'll cover this), kube-proxy sets up the networking rules (using Linux iptables) to route that traffic to the right pod.

**Container Runtime:** The software that actually runs containers. This is usually containerd (which uses Docker's core under the hood). The kubelet tells the container runtime "start this container image."

---

## Concept 1: Pod — The Smallest Runnable Unit

### What it is

A Pod is the smallest thing you can deploy in Kubernetes. It's one or more containers that run together on the same node, share the same network (same IP address), and can share storage.

In 95% of cases, a Pod contains exactly one container. The multi-container case (called "sidecar pattern") is for specific situations like: your app container + a container that collects its logs, or your app container + a container that handles TLS.

### Why it exists (the problem it solves)

Why not just deploy containers directly, like with Docker? Because Kubernetes needs a layer above raw containers to:
- Attach metadata (labels, namespaces)
- Assign resources (CPU/memory limits)
- Handle network identity (one IP per pod, not per container)
- Group tightly-coupled containers that must always run together

### The critical thing to understand about Pods

**Pods are ephemeral. They are cattle, not pets.**

If a pod dies, Kubernetes doesn't "heal" that specific pod — it creates a **new** pod. The new pod has a new name, a new IP address, and no memory of the old pod.

This means you never rely on a pod's IP address or specific name in production. That's what Services are for (we'll get there).

### Hands-on Lab 1: Your First Pod

First, install minikube (local Kubernetes) and kubectl (the CLI tool):

```bash
# Install kubectl — the tool you use to talk to Kubernetes
# Every kubectl command is an API call to the Kubernetes API Server
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# What this does:
# curl -L = follow redirects
# -O = save to file with the remote filename
# $(...) = command substitution: runs the inner curl to get the current stable version string first

chmod +x kubectl
# chmod = change file mode (permissions)
# +x = add execute permission so we can run this file as a program

sudo mv kubectl /usr/local/bin/
# mv = move the file
# /usr/local/bin/ = a standard place for user-installed programs that are available system-wide
# sudo = run as root (administrator), needed because /usr/local/bin/ is owned by root

# Verify kubectl installed
kubectl version --client
# This just checks the local kubectl binary version, doesn't talk to a cluster yet
```

```bash
# Install minikube — runs a single-node Kubernetes cluster on your laptop
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
# install = like cp but also sets permissions correctly

# Start your cluster
minikube start
# This creates a VM (or uses Docker) and sets up a full Kubernetes cluster inside it
# It also configures kubectl to talk to this cluster automatically

# Verify your cluster is running
kubectl cluster-info
# Output will show the API Server URL and other components

kubectl get nodes
# Shows the nodes (machines) in your cluster
# You should see 1 node named "minikube" with status "Ready"
```

Now create your first Pod:

```bash
# The imperative way (quick, but not how you work in production)
kubectl run my-first-pod --image=nginx:latest
# kubectl run = create a pod
# my-first-pod = the pod's name
# --image=nginx:latest = use this container image
# nginx is a web server — it's a common test image because it's lightweight

# Check if it's running
kubectl get pods
# NAME           READY   STATUS    RESTARTS   AGE
# my-first-pod   1/1     Running   0          10s
# 
# READY 1/1 means: 1 container is ready out of 1 total
# STATUS Running = the container is running
# RESTARTS = how many times the container has restarted (0 = healthy so far)

# Get more details
kubectl describe pod my-first-pod
# This shows EVERYTHING: which node it's on, its IP, events (what happened to it), 
# resource limits, environment variables, etc.
# Events at the bottom are your best friend for debugging

# See the logs from your container
kubectl logs my-first-pod
# This is like `docker logs` — shows stdout/stderr from the container

# Execute a command inside the running container
kubectl exec -it my-first-pod -- /bin/bash
# exec = execute a command in a container
# -i = interactive (keeps stdin open)
# -t = allocate a TTY (terminal) — needed for interactive shells
# -- = separates kubectl flags from the command to run inside
# /bin/bash = the shell to open
# Once inside, you're literally inside the container's filesystem. Type 'exit' to leave.

# Delete the pod
kubectl delete pod my-first-pod
# Watch what happens — it's gone. Nothing restarts it.
# This is why we need Deployments.
```

Now the declarative way with a YAML file:

```yaml
# Save this as my-pod.yaml
# YAML is Python-style indentation — spaces matter, no tabs allowed
apiVersion: v1          # Which version of the Kubernetes API to use for this resource
kind: Pod               # What type of resource this is
metadata:
  name: my-nginx-pod    # The pod's name — must be unique in its namespace
  labels:               # Key-value pairs — used to SELECT pods (critical for Services)
    app: nginx
    environment: learning
spec:                   # The desired state — what do you want?
  containers:
  - name: nginx-container        # Name of this container within the pod
    image: nginx:1.25            # Image name:tag — pin versions in production, never use "latest"
    ports:
    - containerPort: 80          # The port your app listens on (documentation only — doesn't restrict access)
    resources:
      requests:                  # The MINIMUM resources reserved for this container
        memory: "64Mi"           # 64 Mebibytes of RAM
        cpu: "100m"              # 100 millicores = 0.1 CPU cores
      limits:                    # The MAXIMUM resources — container is killed if it exceeds this
        memory: "128Mi"
        cpu: "200m"
```

```bash
# Apply the YAML file to your cluster
kubectl apply -f my-pod.yaml
# apply = create or update the resource to match this YAML
# -f = from file
# This is the declarative approach — you describe what you want, K8s makes it happen

# Check it
kubectl get pod my-nginx-pod -o wide
# -o wide = show extra columns like which NODE it's running on and its IP address

# Check pod details including resource usage
kubectl top pod my-nginx-pod
# Shows actual CPU and memory consumption right now
# (requires metrics-server addon: minikube addons enable metrics-server)
```

**What to break intentionally:**
```bash
# Set an impossibly low memory limit
# Edit my-pod.yaml: change limits.memory to "1Mi" (1 megabyte — nginx needs more)
kubectl apply -f my-pod.yaml
kubectl get pods -w  # -w = watch, updates in real-time
# You'll see STATUS go to OOMKilled (Out Of Memory Killed)
# This is how you learn what happens when limits are too tight
# Ctrl+C to stop watching
```

---

## Concept 2: Deployment — Keeping Pods Alive at Scale

### What it is

A Deployment manages a set of identical Pods. You tell the Deployment: "I want 3 replicas of this pod." The Deployment creates 3 pods and watches them. If one dies, the Deployment creates a replacement. If you want to update your app, the Deployment handles the rollout carefully (one at a time, with health checks).

### Why it exists

The raw Pod problem: if your pod dies or you delete it, it's gone. Nothing restarts it. In production, this is unacceptable. You also need to handle:
- **Scaling:** traffic spikes → need more pods; traffic drops → waste less money with fewer pods
- **Rolling updates:** deploy a new version without downtime
- **Rollbacks:** new version is broken → instantly revert to the previous version

The Deployment solves all of this.

### How it works internally

A Deployment doesn't directly manage Pods. It manages a **ReplicaSet**, which manages Pods.

```
Deployment (version 2)
    └── ReplicaSet (v2, 3 replicas)
            ├── Pod-v2-abc
            ├── Pod-v2-def
            └── Pod-v2-ghi
    └── ReplicaSet (v1, 0 replicas) ← kept for rollback history
```

When you update a Deployment, it creates a new ReplicaSet and gradually scales it up while scaling down the old one. This is a **rolling update**. The old ReplicaSet is kept (with 0 pods) so you can roll back by scaling it back up.

### Hands-on Lab 2: Deployments

```yaml
# Save as my-deployment.yaml
apiVersion: apps/v1       # Deployments are in the "apps" API group, not v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3             # I want exactly 3 pods running at all times
  selector:               # HOW does the Deployment know which pods it owns?
    matchLabels:
      app: nginx          # It owns pods that have this label
  template:               # The blueprint for the pods it creates
    metadata:
      labels:
        app: nginx        # MUST match selector.matchLabels above — this connects them
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:   # K8s checks this before sending traffic to the pod
          httpGet:
            path: /       # GET http://pod-ip:80/ — if 200 OK, pod is ready
            port: 80
          initialDelaySeconds: 5   # Wait 5s after container starts before first check
          periodSeconds: 10        # Check every 10 seconds
        livenessProbe:    # K8s checks this to know if it should restart the container
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # At most 1 pod can be unavailable during update
      maxSurge: 1         # At most 1 extra pod can exist during update
```

```bash
kubectl apply -f my-deployment.yaml

# Watch pods get created in real-time
kubectl get pods -w
# You'll see 3 pods appear. Press Ctrl+C when done.

# See the deployment status
kubectl get deployment nginx-deployment
# READY 3/3 = 3 pods ready out of 3 desired
# UP-TO-DATE 3 = 3 pods running the latest pod template
# AVAILABLE 3 = 3 pods available to serve traffic

# The Deployment created a ReplicaSet — see it:
kubectl get replicasets
# Shows the ReplicaSet with 3 desired, 3 current, 3 ready

# Test self-healing: kill a pod
kubectl delete pod <copy-one-pod-name-from-kubectl-get-pods>
# Immediately run:
kubectl get pods
# You'll see the dead pod terminating AND a new pod already creating
# The Deployment noticed the count dropped to 2 and created a replacement
```

Rolling update and rollback:

```bash
# Simulate updating your app to a new version
# Change the image from nginx:1.25 to nginx:1.26
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
# set image = update container image in a deployment
# deployment/nginx-deployment = target this deployment
# nginx=nginx:1.26 = set the "nginx" container's image to nginx:1.26

# Watch the rolling update happen
kubectl rollout status deployment/nginx-deployment
# Shows progress: "Waiting for rollout to finish: 1 out of 3 new replicas have been updated"

# See rollout history
kubectl rollout history deployment/nginx-deployment
# Shows previous versions — this is how rollback works

# Simulate a bad deploy: set a nonexistent image
kubectl set image deployment/nginx-deployment nginx=nginx:this-does-not-exist

# Watch it fail:
kubectl get pods
# You'll see pods stuck in ImagePullBackOff — can't download the image

kubectl rollout status deployment/nginx-deployment
# Will say it's waiting, not finishing

# ROLLBACK! This is the power of Deployments:
kubectl rollout undo deployment/nginx-deployment
# Immediately reverts to the previous ReplicaSet
# Your users never experienced the broken version
```

**Key debugging commands for Deployments:**
```bash
kubectl describe deployment nginx-deployment
# Shows events, conditions, current/desired replica counts

kubectl get events --sort-by='.lastTimestamp'
# All cluster events, sorted by time — good for "what just happened?"
```

---

## Concept 3: StatefulSet — When Pods Need Identity

### What it is

A StatefulSet is like a Deployment, but each Pod has a **stable, persistent identity**: a fixed name, a fixed network hostname, and stable storage that follows it around.

### Why it exists (and why Deployment isn't enough)

Imagine deploying a PostgreSQL database cluster with a Deployment:

- Pod 1 writes data. Gets a name like `postgres-7f9x2`. Has disk attached.
- Pod 1 crashes. Deployment creates `postgres-ab3k1` (new name, possibly new node).
- Problem: The new pod doesn't know it's the "primary" database. It doesn't know which disk has its data. Other pods don't know its new hostname to connect to it.

Stateful apps (databases, message queues, distributed systems like Kafka or Elasticsearch) have requirements that Deployments can't fulfill:

1. **Stable network identity:** `postgres-0`, `postgres-1`, `postgres-2` — always these names, always these DNS hostnames.
2. **Ordered startup/shutdown:** Start `postgres-0` first (primary), then `postgres-1`, `postgres-2` (replicas). Don't start replicas before primary is ready.
3. **Persistent storage per pod:** `postgres-0` always gets its own disk, separate from `postgres-1`'s disk. Even if the pod is rescheduled to a different node, it gets the same storage.

StatefulSet provides all three.

### Key difference to burn into memory

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod names | Random (nginx-7f9x2, nginx-ab3k1) | Ordered (redis-0, redis-1) |
| Startup order | All at once | One by one, in order |
| Storage | Shared or none | Dedicated PVC per pod |
| DNS hostname | Random | Stable (redis-0.redis-svc) |
| Use case | Stateless apps | Databases, Kafka, Zookeeper |

### Hands-on Lab 3: StatefulSet

```yaml
# Save as statefulset-demo.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"     # REQUIRED: Must match a Headless Service (we'll create it)
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www-storage
          mountPath: /usr/share/nginx/html   # Where inside the container to mount storage
  volumeClaimTemplates:   # Creates a SEPARATE PVC for each pod (this is the magic)
  - metadata:
      name: www-storage
    spec:
      accessModes: ["ReadWriteOnce"]   # Only one node can read/write at a time
      resources:
        requests:
          storage: 1Gi
```

```bash
kubectl apply -f statefulset-demo.yaml

kubectl get pods -w
# Watch them come up ONE BY ONE in order:
# web-0 starts first, becomes Running
# Then web-1 starts
# Then web-2 starts
# This is ordered startup — critical for databases

# See the stable names:
kubectl get pods
# web-0, web-1, web-2 — always these names

# Each pod got its own storage
kubectl get pvc  
# pvc = PersistentVolumeClaim
# You'll see: www-storage-web-0, www-storage-web-1, www-storage-web-2
# Each pod's data is separate and persistent

# Prove stable DNS: exec into web-0 and ping web-1 by hostname
kubectl exec -it web-0 -- /bin/bash
# Inside the container:
apt-get update && apt-get install -y dnsutils  # install DNS tools
nslookup web-1.web.default.svc.cluster.local   # stable DNS for web-1
# Format: <pod-name>.<service-name>.<namespace>.svc.cluster.local
exit
```

---

## Concept 4: DaemonSet — One Pod Per Node

### What it is

A DaemonSet ensures that exactly one copy of a Pod runs on every Node in the cluster. When you add a new node, the DaemonSet automatically puts a pod on it. When you remove a node, the pod is garbage collected.

### Why it exists

Some things need to run everywhere: log collectors, metrics agents, security scanners, network plugins. You don't want to manage "run this on node 1, 2, 3..." manually. And when your cluster grows from 10 to 100 nodes, you don't want to manually add agents to each new node.

Real examples from production:
- **Fluentd/Filebeat:** Log collector that reads container logs from each node's filesystem and ships them to Elasticsearch
- **Prometheus Node Exporter:** Collects OS-level metrics (CPU, disk, network) from each node
- **Datadog Agent:** APM and monitoring agent
- **Calico/Cilium:** Network plugin (actually manages Pod networking)

### Hands-on Lab 4: DaemonSet

```yaml
# Save as daemonset-demo.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
  labels:
    app: node-logger
spec:
  selector:
    matchLabels:
      app: node-logger
  template:
    metadata:
      labels:
        app: node-logger
    spec:
      containers:
      - name: logger
        image: busybox:latest   # Tiny Linux image, good for demos
        command: ["/bin/sh", "-c", "while true; do echo 'Collecting logs from node'; sleep 30; done"]
        # This simulates a log collector that runs forever
        resources:
          requests:
            memory: "16Mi"
            cpu: "10m"
```

```bash
kubectl apply -f daemonset-demo.yaml

kubectl get daemonsets
# DESIRED = number of nodes (should be 1 in minikube)
# CURRENT = pods currently running
# READY = pods that are ready

kubectl get pods -l app=node-logger -o wide
# -l app=node-logger = filter by label
# -o wide = show which node each pod is on
# You'll see each pod is on a different node (with multiple nodes)
```

---

## Concept 5: Service — Stable Network Endpoint

### What it is

A Service is a stable IP address and DNS name that routes traffic to a set of Pods matching specific labels.

### Why it exists — and this is critical

Pods are ephemeral. They die, get replaced, get new IPs. You cannot hardcode a Pod's IP address anywhere. A Service gives you a stable "front door" that always points to the right pods, regardless of which specific pods are alive right now.

```
Before Service:                After Service:
                               
Client → 10.244.1.5 (Pod A)   Client → my-service:80 (stable!)
         10.244.1.5 dies               ↓
Client → ??? broken!           Service → 10.244.2.7 (Pod A replacement)
                                      → 10.244.1.8 (Pod B)
                                      → 10.244.3.2 (Pod C)
```

The Service uses **label selectors** to know which pods to route to. The same mechanism as Deployments — it's all labels.

### Types of Services

**ClusterIP (default):** Only accessible from within the cluster. Pod A to Pod B. No external access. 99% of internal services use this.

**NodePort:** Opens a port (30000-32767) on every node. Traffic to `<any-node-ip>:<nodeport>` reaches your service. Useful for development, not recommended for production (exposes your nodes directly).

**LoadBalancer:** In cloud environments (AWS, GCP, Azure), creates an external load balancer. The cloud gives you an external IP. Your users access this IP. This is how you expose apps to the internet in production.

**Headless Service (ClusterIP: None):** No stable IP. Instead, DNS returns the IPs of all individual pods. Used with StatefulSets so each pod gets its own DNS entry.

### Hands-on Lab 5: Services

```yaml
# Save as my-service.yaml
# This assumes nginx-deployment from Lab 2 is still running
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx       # Route to pods that have this label — same labels as our Deployment's pods
  ports:
  - protocol: TCP
    port: 80         # The port THIS SERVICE listens on (inside the cluster)
    targetPort: 80   # The port on the POD to forward traffic to
  type: ClusterIP    # Only accessible within the cluster
```

```bash
kubectl apply -f my-service.yaml

kubectl get service nginx-service
# NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# nginx-service   ClusterIP   10.96.45.123    <none>        80/TCP    5s
# CLUSTER-IP is the stable IP — this won't change

# See which pods this service will route to
kubectl get endpoints nginx-service
# Shows the actual pod IPs behind the service
# If no pods match the selector, this will be empty (common debugging scenario!)

# Test the service from inside the cluster
kubectl run test-pod --image=busybox --rm -it -- /bin/sh
# --rm = delete this pod when you exit
# Inside the shell:
wget -qO- nginx-service    # Access by service NAME — Kubernetes DNS resolves it
wget -qO- nginx-service.default.svc.cluster.local  # Full DNS name
# Both should return the nginx welcome page
exit

# Expose externally for testing (NodePort)
kubectl expose deployment nginx-deployment --type=NodePort --port=80
# Creates a new service that opens a random port on your node

kubectl get service nginx-deployment
# PORT(S) will show something like 80:31234/TCP
# 31234 is the NodePort

minikube service nginx-deployment --url
# minikube gives you the URL to access this in your browser
```

**The crucial debugging skill — when a Service isn't working:**
```bash
# Step 1: Does the service exist and have the right selector?
kubectl describe service nginx-service

# Step 2: Are there endpoints? (Do any pods match the selector?)
kubectl get endpoints nginx-service
# Empty endpoints = no pods match the selector labels — check your labels!

# Step 3: Are the pods actually running?
kubectl get pods -l app=nginx

# Step 4: Check pod logs for application errors
kubectl logs <pod-name>

# This 4-step process solves 90% of service connectivity issues
```

---

## Concept 6: Ingress — HTTP Routing to the World

### What it is

An Ingress is an API object that manages external HTTP/HTTPS access to Services. It provides: hostname-based routing, path-based routing, TLS termination (HTTPS), and can handle many services through a single entry point.

### Why it exists

Without Ingress, to expose 5 different services to the internet, you'd need 5 LoadBalancers — 5 external IPs, 5 cloud load balancers, 5 separate bills.

With Ingress, you have ONE load balancer, ONE external IP, and it routes based on the URL:

```
External Traffic → Ingress Controller (1 LoadBalancer, 1 IP)
                        ├── app.example.com → frontend-service
                        ├── api.example.com → backend-service
                        ├── app.example.com/admin → admin-service
                        └── app.example.com/static → storage-service
```

### Important: Ingress needs an Ingress Controller

The Ingress object itself is just configuration — a YAML file that says "route this hostname to that service." For it to actually work, you need an **Ingress Controller** running in the cluster: software that reads Ingress objects and actually configures the routing.

Common controllers: **nginx-ingress**, **Traefik**, **AWS ALB Ingress Controller**, **GKE Ingress**.

### Hands-on Lab 6: Ingress

```bash
# Enable the nginx ingress controller in minikube
minikube addons enable ingress

# Verify the ingress controller pod is running
kubectl get pods -n ingress-nginx
# -n ingress-nginx = look in the ingress-nginx namespace
# Wait until the controller pod shows Running
```

```yaml
# Save as my-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /   # Strip path prefix before forwarding
spec:
  rules:
  - host: myapp.local         # Route requests for this hostname
    http:
      paths:
      - path: /nginx           # When URL starts with /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service  # Forward to this service
            port:
              number: 80
```

```bash
kubectl apply -f my-ingress.yaml

kubectl get ingress
# Shows the ingress with its ADDRESS (the ingress controller's IP)

# Get the minikube IP
minikube ip

# Add to /etc/hosts (the system's local DNS override file)
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts
# tee = write to both stdout and the file
# -a = append (don't overwrite)
# /etc/hosts = DNS lookup happens here BEFORE asking real DNS servers
# This lets us fake a real domain name locally

curl http://myapp.local/nginx
# Should return nginx's welcome page
```

---

## Concept 7: ConfigMap — Non-Secret Configuration

### What it is

A ConfigMap stores configuration data as key-value pairs. Your app reads this config instead of having it hardcoded in the container image.

### Why it exists

The golden rule of containers: **Build once, run anywhere.** Your container image should be identical whether it runs in development, staging, or production. The ONLY thing that should change between environments is configuration.

Without ConfigMap, you'd need different Docker images for different environments, or bake secrets into images (terrible security).

With ConfigMap:
- One image
- Different ConfigMap in dev vs staging vs prod
- App reads config at runtime

Examples of what goes in ConfigMaps: database hostnames, feature flags, log levels, API endpoint URLs, nginx.conf files, application.properties.

### Hands-on Lab 7: ConfigMap

```yaml
# Save as my-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Simple key-value pairs
  LOG_LEVEL: "info"
  DATABASE_HOST: "postgres.default.svc.cluster.local"
  MAX_CONNECTIONS: "100"
  
  # You can store entire file contents as a value
  nginx.conf: |
    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
        }
    }
```

```bash
kubectl apply -f my-configmap.yaml

kubectl get configmap app-config
kubectl describe configmap app-config
# Shows all keys and their values
```

Using ConfigMap in a Pod — two methods:

```yaml
# Method 1: Inject as environment variables
spec:
  containers:
  - name: myapp
    image: nginx:1.25
    envFrom:
    - configMapRef:
        name: app-config    # All keys become environment variables
    # OR inject specific keys:
    env:
    - name: LOG_LEVEL       # env var name in the container
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL    # The key in the ConfigMap

# Method 2: Mount as files (use for config files like nginx.conf)
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d/   # Mount directory inside container
  volumes:
  - name: config-volume
    configMap:
      name: app-config               # The ConfigMap to mount
      items:
      - key: nginx.conf              # This specific key
        path: default.conf           # becomes this filename
```

**Critical limitation:** Changes to a ConfigMap don't automatically restart pods. Pods read the ConfigMap at startup (for env vars) or periodically (for volume mounts, with ~60s delay). In production, you often need to restart pods after config changes. Some teams add a hash of the ConfigMap to the pod template annotations to force restarts.

---

## Concept 8: Secret — Sensitive Configuration

### What it is

A Secret is like a ConfigMap but for sensitive data: passwords, API tokens, TLS certificates, SSH keys. Values are base64-encoded (NOT encrypted by default — this is a common misconception).

### The Security Reality

**base64 is NOT encryption.** It's just encoding. `echo "mypassword" | base64` gives `bXlwYXNzd29yZA==`. Anyone can decode it.

Real security for Secrets comes from:
- **RBAC:** Only specific service accounts and users can read specific secrets
- **Encryption at rest:** Configure etcd to encrypt secrets (disabled by default, must be explicitly enabled)
- **Sealed Secrets or Vault:** External secret management tools used in production
- **Never committing Secrets to Git** — this is how most credential leaks happen

Kubernetes Secrets exist to separate sensitive config from pods (so you don't hardcode passwords in images) and to give Kubernetes RBAC control over who can access them.

### Hands-on Lab 8: Secrets

```bash
# Create a secret imperatively (for quick creation)
kubectl create secret generic db-credentials \
  --from-literal=username=dbuser \
  --from-literal=password=supersecretpassword
# --from-literal = create key-value from this string
# generic = the type (others: tls, docker-registry)

# See the secret
kubectl get secret db-credentials
# Notice: the Data column shows the NUMBER of keys, not the values

# Describe it
kubectl describe secret db-credentials
# Still no values — Kubernetes won't show secret values in describe

# Get the actual values (base64 encoded)
kubectl get secret db-credentials -o yaml
# You'll see: password: c3VwZXJzZWNyZXRwYXNzd29yZA==
# Decode it:
echo "c3VwZXJzZWNyZXRwYXNzd29yZA==" | base64 --decode
# Outputs: supersecretpassword
# This proves base64 is NOT security — anyone with kubectl access can decode it
```

```yaml
# Using a secret in a pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: DB_PASSWORD      # Environment variable name inside the container
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    # OR mount as a file (more secure — file permissions can restrict access)
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true         # Container can only read, not write
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      # Creates files: /etc/secrets/username and /etc/secrets/password
```

---

## Concept 9: Namespace — Cluster Isolation

### What it is

A Namespace is a virtual cluster within a cluster. It's a boundary that isolates resources. Resources in the same namespace can communicate by service name. Resources across namespaces need the full DNS name.

### Why it exists

A single Kubernetes cluster at a company might serve: the payments team, the frontend team, the data engineering team, and 3 environments (dev, staging, prod). Without namespaces, everything is in one flat space. Names would clash. Team A could accidentally delete Team B's resources. There'd be no way to set different resource quotas.

Namespaces enable:
- **Isolation:** `kubectl delete all --all` in your namespace won't touch other teams
- **Resource quotas:** Dev namespace gets 100GB memory max; prod gets 2TB
- **RBAC scope:** Give a team admin rights to their namespace, not the whole cluster
- **Organization:** `kubectl get pods -n payments` shows only payments team pods

### Built-in namespaces

- `default` — where your stuff goes if you don't specify a namespace
- `kube-system` — Kubernetes internal components (don't touch)
- `kube-public` — publicly readable data (rarely used)
- `kube-node-lease` — node heartbeat data (internal)

### Hands-on Lab 9: Namespaces

```bash
# See existing namespaces
kubectl get namespaces

# Create a namespace for your app
kubectl create namespace my-app-dev

# Create another for staging
kubectl create namespace my-app-staging

# Deploy to a specific namespace
kubectl apply -f my-deployment.yaml -n my-app-dev
# -n my-app-dev = in this namespace

# See all pods across ALL namespaces
kubectl get pods --all-namespaces
# or shorthand:
kubectl get pods -A

# See kube-system namespace (Kubernetes internals)
kubectl get pods -n kube-system
# You'll see coredns (cluster DNS), kube-proxy, the ingress controller, etc.

# Set a default namespace for your current context (so you don't type -n every time)
kubectl config set-context --current --namespace=my-app-dev
# Now all commands default to my-app-dev

# Switch back to default
kubectl config set-context --current --namespace=default
```

Resource quota for a namespace:

```yaml
# Save as namespace-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: my-app-dev
spec:
  hard:
    requests.cpu: "2"           # Total CPU requests across all pods in this namespace
    requests.memory: 2Gi        # Total memory requests
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "20"                  # Max number of pods
    persistentvolumeclaims: "5" # Max number of PVCs
```

```bash
kubectl apply -f namespace-quota.yaml

kubectl describe namespace my-app-dev
# Shows the quota and current usage
```

---

## Concept 10: Node — The Worker Machine

### What it is

A Node is a physical or virtual machine that runs Pods. The Control Plane decides which pods go where; the Node actually runs them.

### What runs on a Node

**kubelet:** The agent that communicates with the API Server. It receives pod specs ("run this container") and tells the container runtime to start them. It also runs health checks (liveness/readiness probes) and reports pod status back.

**kube-proxy:** Manages network rules on the node. When you create a Service, kube-proxy creates iptables rules on every node so that traffic to the Service IP gets routed to the right pods.

**Container Runtime:** Usually containerd. The software that actually runs containers. kubelet tells containerd "start this image", containerd starts the container.

### Hands-on Lab 10: Nodes

```bash
# See all nodes
kubectl get nodes

# Detailed node info: capacity, allocated resources, conditions
kubectl describe node minikube
# Look for:
# Capacity: total CPU and memory on the machine
# Allocatable: what's available for pods (some is reserved for system)
# Conditions: Ready/NotReady, DiskPressure, MemoryPressure
# Non-terminated Pods: all pods currently on this node

# Check node resource usage
kubectl top nodes
# Shows actual CPU and memory consumption

# Cordon a node (stop scheduling NEW pods on it, but don't evict existing ones)
kubectl cordon minikube
# Used before draining — marks node unschedulable

# Drain a node (evict all pods for maintenance/decommissioning)
kubectl drain minikube --ignore-daemonsets --delete-emptydir-data
# --ignore-daemonsets = don't evict DaemonSet pods (they're supposed to be everywhere)
# --delete-emptydir-data = evict pods that use emptyDir volumes (ephemeral storage)

# Uncordon to bring back into service
kubectl uncordon minikube
```

**Taints and Tolerations (important production concept):**

A taint marks a node: "Don't schedule pods here unless they explicitly tolerate this taint." Used to dedicate nodes for specific workloads.

```bash
# Add a taint: only GPU workloads should run on GPU nodes
kubectl taint nodes minikube gpu=true:NoSchedule
# NoSchedule = new pods without a matching toleration won't be scheduled here

# Now your GPU pod needs this toleration in its spec:
# tolerations:
# - key: "gpu"
#   operator: "Equal"
#   value: "true"
#   effect: "NoSchedule"

# Remove the taint
kubectl taint nodes minikube gpu=true:NoSchedule-
# The trailing - removes the taint
```

---

## Concept 11: Control Plane — The Brain

### What it is

The Control Plane is the set of components that make global decisions about the cluster. It's the brain — it watches the desired state and works to achieve it.

### The Reconciliation Loop (Core Concept)

Every controller in Kubernetes runs a reconciliation loop:

```
1. Read desired state from etcd (e.g., "I want 3 replicas")
2. Observe current state (e.g., "2 pods are running")
3. Take action to close the gap (e.g., create 1 more pod)
4. Go back to step 1, repeat forever
```

This is called **level-triggered** (not event-triggered). It doesn't matter how we got to the current state — it only matters what the current state is vs the desired state. This makes Kubernetes self-healing by design.

### The Flow of `kubectl apply`

When you run `kubectl apply -f my-deployment.yaml`, here's the exact sequence:

```
1. kubectl parses YAML, sends HTTP POST to API Server
2. API Server authenticates (is this user allowed?)
3. API Server authorizes (is this user allowed to create Deployments?)
4. API Server validates (is the YAML valid? Are required fields present?)
5. API Server writes to etcd ("desired state: 3 nginx pods")
6. API Server returns 201 Created to kubectl
7. Deployment Controller (in Controller Manager) notices a new Deployment
8. Deployment Controller creates a ReplicaSet
9. ReplicaSet Controller notices the ReplicaSet needs 3 pods
10. ReplicaSet Controller creates 3 Pod objects (writes to etcd — no pods running yet)
11. Scheduler notices 3 pods with no node assigned
12. Scheduler picks nodes based on resource availability, writes node assignments to etcd
13. kubelet on each chosen node notices pods assigned to it
14. kubelet tells containerd to pull the image and start the container
15. kubelet reports back: "Pod X is Running" — writes to etcd
16. ReplicaSet Controller confirms: 3/3 running. ✓
```

This is why Kubernetes is powerful — every component does one thing and communicates only through the API Server and etcd.

---

## Concept 12: RBAC — Who Can Do What

### What it is

RBAC (Role-Based Access Control) controls who (users, services, pods) can do what (create, read, update, delete) to which resources (pods, secrets, deployments) in which namespaces.

### Why it exists

You don't want every pod to be able to read all Secrets. You don't want a junior developer to accidentally delete a production Deployment. You don't want one microservice to be able to talk to another microservice's secrets.

The principle of **least privilege**: grant only the permissions needed to do the job, nothing more.

### The RBAC model

```
Subject (who)          Verb (what)         Resource (on what)
User/Group/            get, list,          pods, secrets,
ServiceAccount   →     create, update,  →  deployments, services
                       delete, patch        in a namespace or cluster-wide
```

**Role:** Defines permissions within a namespace  
**ClusterRole:** Defines permissions cluster-wide  
**RoleBinding:** Grants a Role to a Subject in a namespace  
**ClusterRoleBinding:** Grants a ClusterRole to a Subject cluster-wide  

### Hands-on Lab 12: RBAC

```yaml
# Save as rbac-demo.yaml
# Scenario: A CI/CD pipeline needs to deploy (create/update) pods, 
# but should NEVER be able to read secrets

# Step 1: Create a ServiceAccount for the CI pipeline
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-pipeline
  namespace: default
---
# Step 2: Define WHAT it can do (Role)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployer
  namespace: default
rules:
- apiGroups: ["apps"]              # "apps" group contains Deployments
  resources: ["deployments"]       # On Deployments
  verbs: ["get", "list", "create", "update", "patch"]  # Can do these things
- apiGroups: [""]                  # "" = core API group (pods, services, etc.)
  resources: ["pods"]
  verbs: ["get", "list"]           # Can only READ pods, not create/delete them directly
# Notice: no rules for "secrets" — denied by default
---
# Step 3: Bind the Role to the ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-pipeline-deployer
  namespace: default
subjects:
- kind: ServiceAccount
  name: ci-pipeline      # Grant to this ServiceAccount
  namespace: default
roleRef:
  kind: Role
  name: deployer          # This Role
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rbac-demo.yaml

# Test what the service account can do
kubectl auth can-i get pods --as=system:serviceaccount:default:ci-pipeline
# Output: yes

kubectl auth can-i get secrets --as=system:serviceaccount:default:ci-pipeline
# Output: no  ← RBAC working correctly

kubectl auth can-i delete deployments --as=system:serviceaccount:default:ci-pipeline
# Output: no

# List all permissions for a service account
kubectl auth can-i --list --as=system:serviceaccount:default:ci-pipeline
```

---

## Putting It All Together: Full Application Deployment

Now let's deploy a complete application using everything we've learned: a backend API + Redis cache, with Services, ConfigMaps, Secrets, and Ingress.

```yaml
# Save as full-app.yaml
# This deploys: Redis + a simple web app that uses Redis

---
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: my-app
---
# Secret for Redis password
apiVersion: v1
kind: Secret
metadata:
  name: redis-secret
  namespace: my-app
type: Opaque
data:
  password: cmVkaXNwYXNzd29yZA==   # base64 of "redispassword"
---
# ConfigMap for app config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: my-app
data:
  REDIS_HOST: "redis-service"       # Use service name, not IP!
  REDIS_PORT: "6379"
  LOG_LEVEL: "info"
---
# Redis StatefulSet (it's stateful — stores data)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: my-app
spec:
  serviceName: "redis-service"
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: password
        command: ["redis-server", "--requirepass", "$(REDIS_PASSWORD)"]
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "256Mi"
            cpu: "200m"
---
# Redis Service (internal only — ClusterIP)
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: my-app
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
---
# Web App Deployment (stateless — use Deployment)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:1.25   # Pretend this is your app
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: app-config
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
---
# Web App Service
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: my-app
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f full-app.yaml

# Watch everything come up
kubectl get all -n my-app
# Shows all resources in the namespace

# Verify the full chain:
# 1. Are pods running?
kubectl get pods -n my-app

# 2. Do services have endpoints?
kubectl get endpoints -n my-app

# 3. Can the web app reach Redis?
kubectl exec -it -n my-app deployment/web-app -- /bin/sh
# curl redis-service:6379 or ping redis-service
exit
```

---

## Debugging Methodology: The Systematic Approach

When something breaks in Kubernetes, use this flow every time. Don't guess — be systematic.

### The Debugging Ladder

```
1. Start at the TOP (Deployment/StatefulSet) and work DOWN
   kubectl get deployment <name>
   → If READY shows 0/3 or similar, something's wrong with pods

2. Look at the Pods
   kubectl get pods
   → What's the STATUS? (Pending? CrashLoopBackOff? ImagePullBackOff? OOMKilled?)

3. Describe the Pod (most useful command in Kubernetes)
   kubectl describe pod <pod-name>
   → EVENTS section at the bottom is the goldmine
   → Look for: Failed scheduling, Failed pulling image, Back-off restarting

4. Check logs
   kubectl logs <pod-name>
   kubectl logs <pod-name> --previous   # logs from the PREVIOUS container run (if it crashed)
   kubectl logs <pod-name> -f           # follow in real-time

5. For network issues: check Service and Endpoints
   kubectl get endpoints <service-name>
   → Empty endpoints = no pods match the selector

6. Exec in for live debugging
   kubectl exec -it <pod-name> -- /bin/sh
   → Can I reach the database? Can I reach other services?
   → Check env vars: env | grep DATABASE
```

### Common Status Messages and What They Mean

| Status | What it means | How to fix |
|--------|--------------|------------|
| `Pending` | Pod created but not scheduled | kubectl describe pod — check Events for "Insufficient memory" or "no nodes available" |
| `CrashLoopBackOff` | Container starts, crashes, K8s keeps restarting | kubectl logs --previous — look at the error |
| `ImagePullBackOff` | Can't pull container image | Wrong image name? Wrong tag? Private registry without credentials? |
| `OOMKilled` | Out of Memory — container exceeded memory limit | Increase memory limit or fix memory leak |
| `Evicted` | Node ran out of resources and evicted this pod | Move to a larger node or reduce resource usage |
| `Terminating` (stuck) | Pod won't die gracefully | kubectl delete pod <name> --force --grace-period=0 |

---

## Quick Reference: Essential kubectl Commands

```bash
# VIEWING RESOURCES
kubectl get pods                          # List pods in current namespace
kubectl get pods -A                       # List pods in ALL namespaces  
kubectl get pods -o wide                  # With node IP info
kubectl get pods -w                       # Watch for changes (real-time)
kubectl get all                           # Get pods, services, deployments, etc.
kubectl get all -n <namespace>            # Same but for specific namespace

# INSPECTING RESOURCES
kubectl describe pod <name>               # Full details + events
kubectl logs <pod>                        # Container logs
kubectl logs <pod> -c <container>         # Specific container in multi-container pod
kubectl logs <pod> --previous             # Logs from crashed previous container
kubectl logs <pod> -f                     # Follow/stream logs

# DEBUGGING
kubectl exec -it <pod> -- /bin/bash       # Open shell in container
kubectl exec -it <pod> -- env             # List env vars without opening shell
kubectl top pods                          # Resource usage
kubectl get events --sort-by=.lastTimestamp  # Recent cluster events

# APPLYING AND MANAGING
kubectl apply -f file.yaml                # Create or update resource
kubectl delete -f file.yaml              # Delete resources defined in file
kubectl delete pod <name>                # Delete specific pod
kubectl scale deployment <name> --replicas=5  # Scale

# ROLLING UPDATES
kubectl set image deployment/<name> <container>=<newimage>  # Update image
kubectl rollout status deployment/<name>  # Watch rollout progress
kubectl rollout history deployment/<name> # See rollout history
kubectl rollout undo deployment/<name>    # Rollback

# NAMESPACES
kubectl get pods -n <namespace>           # Resources in namespace
kubectl config set-context --current --namespace=<ns>  # Set default namespace
kubectl create namespace <name>           # Create namespace

# CONTEXT (which cluster you're talking to)
kubectl config get-contexts               # List all clusters/contexts
kubectl config use-context <name>         # Switch cluster
kubectl config current-context            # Show current context
```

---

## The Mental Map: What to Use When

```
Your app has no state? (web servers, APIs)
→ Deployment

Your app has state? (databases, Kafka, Elasticsearch)
→ StatefulSet

Need something on every node? (logging, monitoring, security agents)
→ DaemonSet

Need to expose pods inside the cluster?
→ Service (ClusterIP)

Need to expose a service to the internet (one service)?
→ Service (LoadBalancer)

Need to expose MULTIPLE services with routing rules / HTTPS?
→ Ingress

Need to store non-sensitive config?
→ ConfigMap

Need to store sensitive data (passwords, tokens)?
→ Secret

Need to separate teams/environments?
→ Namespace

Need to control who can do what?
→ RBAC (Roles, ClusterRoles, Bindings)
```

---

## What to Study Next (In Order)

1. **Persistent Volumes (PV/PVC):** How StatefulSets actually store data, storage classes, dynamic provisioning
2. **Resource Quotas and LimitRanges:** Preventing namespace resource abuse
3. **Horizontal Pod Autoscaler (HPA):** Auto-scaling pods based on CPU/memory
4. **Network Policies:** Firewall rules between pods — by default all pods can talk to all pods
5. **Helm:** Package manager for Kubernetes — instead of managing 10 YAML files, package them
6. **Operators:** Custom controllers for complex stateful apps (databases, cert-manager)
7. **Prometheus + Grafana:** Monitoring your cluster and apps
8. **CI/CD with Kubernetes:** GitLab CI/GitHub Actions deploying to K8s

---

## Self-Test: Can You Explain These Without Notes?

After reading each section, close this doc and try to answer:

1. What problem did containers (Docker) solve? What problem does Kubernetes add on top?
2. What is the difference between a Pod and a container?
3. What happens when a Pod in a Deployment crashes?
4. When do you use a StatefulSet instead of a Deployment? Give a real example.
5. Why do Pods need Services? What problem does the ephemeral IP address create?
6. What is the difference between ClusterIP, NodePort, and LoadBalancer?
7. What does an Ingress do that a Service can't do alone?
8. What is the difference between a ConfigMap and a Secret?
9. Why is base64 not encryption?
10. What does RBAC stand for and what principle does it implement?
11. Explain the reconciliation loop. How does it make Kubernetes self-healing?
12. What runs on a Node? What does kubelet actually do?
13. Walk through what happens when you run `kubectl apply -f deployment.yaml`.

If you can answer all 13 fluently, you understand Kubernetes. If you stumble on any, reread that section.

---

*Last updated: 2026-02-20 | Study this daily until the concepts feel like common sense, not memorization.*
