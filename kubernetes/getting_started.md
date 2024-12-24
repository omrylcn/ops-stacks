# Kubernetes Core Concepts Guide

## Understanding Kubernetes Architecture

Kubernetes is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a framework for running distributed systems resiliently.

### Control Plane

The control plane manages the Kubernetes cluster. It's responsible for making global decisions about the cluster (e.g., scheduling), as well as detecting and responding to cluster events.

1. **API Server (kube-apiserver):** The API server is the front end for the Kubernetes control plane. All interactions with the cluster go through it. It exposes the Kubernetes API, which allows users, other control plane components, and external systems to communicate with the cluster.

2. **etcd:** etcd is a consistent and highly-available key-value store used as Kubernetes' backing store for cluster data. It stores the configuration, state, and metadata of the cluster. *Backing up etcd is crucial for disaster recovery.*

    ```bash
    # Backup etcd data (adjust paths and endpoints as needed)
    ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save backup.db
    ```

3. **Controller Manager (kube-controller-manager):** The controller manager runs various controller processes. Each controller is responsible for a specific aspect of the cluster's state. For example, the Deployment controller manages Deployments, the ReplicaSet controller manages ReplicaSets, and the Node controller manages Nodes. Controllers work by continuously comparing the desired state of the cluster with its current state and taking actions to reconcile any differences. This process is called the *control loop*.

4. **Scheduler (kube-scheduler):** The scheduler is responsible for assigning Pods to Nodes. When you create a Pod, the scheduler finds the most suitable Node to run it on, considering factors like resource availability, node affinity, and taints/tolerations.

### Worker Nodes

Worker nodes are the machines where your applications actually run.

1. **Kubelet (kubelet):** The kubelet is an agent that runs on each Node. It receives instructions from the API server and ensures that containers are running in Pods as specified.

    ```bash
    # See all nodes and their status
    kubectl get nodes
    ```

2. **Container Runtime:** The container runtime is responsible for running containers. Common container runtimes include Docker, containerd, and CRI-O.

    ```bash
    # Check what container engine a node uses
    kubectl describe node <node-name> | grep "Container Runtime Version"
    ```

3. **Kube-Proxy (kube-proxy):** kube-proxy is a network proxy that runs on each Node. It implements Kubernetes Services, which provide network access to Pods. It maintains network rules on Nodes, allowing network communication to your Pods from inside or outside of your cluster.

## Basic Concepts

### Pods

A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster. A Pod can contain one or more containers that share network and storage resources.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.23 # Use a more recent version
    ports:
    - containerPort: 80
    resources: # Add resource requests and limits
      requests:
        cpu: 100m
        memory: 200Mi
      limits:
        cpu: 200m
        memory: 400Mi
```

```bash
# Create pod
kubectl apply -f pod.yaml # Use apply for declarative management

# Get pods
kubectl get pods

# Describe pod (for detailed information and events)
kubectl describe pod nginx-pod

# Delete pod
kubectl delete pod nginx-pod
```

### YAML Structure

```yaml
apiVersion: <group>/<version> # e.g., apps/v1, v1
kind: <Resource> # e.g., Pod, Deployment, Service
metadata:
  name: <name>
  namespace: <namespace> # Important for resource organization
  labels:
    <key>: <value> # Used for selectors and organization
spec: # Desired state of the resource
  # ... resource-specific configuration ...
status: # Current state of the resource (managed by Kubernetes)
  # ... status information ...
```

## Management Approaches

### Declarative (Recommended)

You define the desired state of your system in YAML files, and Kubernetes works to achieve and maintain that state. This is the preferred approach for production environments.

```bash
kubectl apply -f <file.yaml> # Creates or updates resources
kubectl apply -f ./configs/ # Applies all YAML files in a directory
```

### Imperative

You use direct `kubectl` commands to perform actions. This is useful for quick tasks and experimentation but less suitable for managing complex deployments.

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

## Tools and Setup

### Minikube

Minikube is a tool that runs a single-node Kubernetes cluster inside a VM on your local machine. It's great for development and testing.

```bash
minikube start
minikube status
minikube dashboard
minikube stop
minikube delete
```

### kind (Kubernetes IN Docker)

kind uses Docker containers as "nodes." This is a lightweight and fast way to run local Kubernetes clusters.

### Docker Desktop with Kubernetes

Docker Desktop also provides an integrated Kubernetes environment.

### kubectl

`kubectl` is the command-line tool for interacting with Kubernetes clusters.

```bash
kubectl config view # View your kubeconfig file
kubectl config use-context <context-name>
kubectl config current-context
kubectl get <resource> # e.g., kubectl get pods, kubectl get deployments
kubectl describe <resource> <name>
kubectl create <resource> -f <file.yaml>
kubectl delete <resource> <name>
```

## Workload Controllers

### ReplicaSet

A ReplicaSet ensures that a specified number of Pod replicas are running at all times. If a Pod crashes, the ReplicaSet creates a new one.

### Deployment

A Deployment manages ReplicaSets and provides declarative updates for Pods and ReplicaSets. It supports rolling updates and rollbacks. *Deployments are the recommended way to manage stateless applications in Kubernetes.*

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
  template: # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: 200m
            memory: 400Mi
```

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.24
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

### StatefulSet

StatefulSets are used for managing stateful applications that require stable network IDs, persistent storage, and ordered deployment and scaling.

## Networking

### Service

A Service exposes an application running on a set of Pods as a network service. It provides a stable IP address and DNS name for accessing the Pods.

#### Service Types

1. **ClusterIP:** Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster. (Default)
2. **NodePort:** Exposes the Service on each Node's IP at a static port (the NodePort). You can contact the NodePort from outside the cluster.
3. **LoadBalancer:** Exposes the Service externally using a cloud provider's load balancer.
4. **ExternalName:** Maps the Service to an external DNS name.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80 # Service port
    targetPort: 80 # Container port
```

```bash
kubectl expose deployment nginx --port=80 --type=ClusterIP #or NodePort or LoadBalancer
kubectl get services
kubectl describe service nginx-service
```

### Ingress

An Ingress exposes HTTP and HTTPS routes from outside the cluster to Services within
