# Kubernetes Namespaces Guide

## Overview

Namespaces provide a mechanism for isolating groups of resources within a single cluster. They are intended for use in environments with many users spread across multiple teams or projects.

## Default Namespaces

### 1. default

- Default namespace for objects with no other namespace specified
- Used when no namespace is explicitly set
- Contains all user-created resources by default

### 2. kube-system

- Reserved for Kubernetes system resources
- Contains core components like:
  - CoreDNS
  - kube-proxy
  - Controller Manager
  - API Server
  - etcd
  - Scheduler

### 3. kube-public

- Automatically created
- Readable by all users (including unauthenticated)
- Reserved for cluster usage
- Contains public information about the cluster

### 4. kube-node-lease

- Holds node lease objects
- Used for node heartbeat mechanism
- Improves scalability of node status updates

## Namespace Operations

### Basic Commands

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns

# Get detailed namespace information
kubectl get namespace -o wide
kubectl describe namespace my-namespace

# Create namespace
kubectl create namespace my-namespace
kubectl create ns my-namespace

# Delete namespace
kubectl delete namespace my-namespace
kubectl delete ns my-namespace

# Set default namespace for current context
kubectl config set-context --current --namespace=my-namespace

# View resources in a namespace
kubectl get all -n my-namespace
```

### YAML Definitions

#### Create Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
    environment: dev
```

#### Pod in Specific Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: development
spec:
  containers:
  - name: nginx
    image: nginx
```

## Use Cases

### 1. Team Isolation

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-alpha
  labels:
    team: alpha
    environment: production
```

### 2. Environment Separation

```yaml
# Development namespace
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: dev

---
# Production namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: prod
```

### 3. Resource Management

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
```

### 4. Access Control

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-access
  namespace: development
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

## Namespace Features

### 1. Resource Isolation

- Resources in different namespaces can have the same name
- Network policies can be namespace-specific
- Service discovery within namespace

### 2. Resource Quotas

- CPU limits
- Memory limits
- Storage quotas
- Object count limits

### 3. Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-access
  namespace: development
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
```

### 4. Role-Based Access Control (RBAC)

- Namespace-specific roles
- Service accounts
- Role bindings

## Best Practices

1. **Naming Conventions**
   - Use descriptive names
   - Follow consistent patterns
   - Include environment or team identifiers

2. **Resource Organization**

   ```yaml
   metadata:
     namespace: team-project-environment
     labels:
       team: frontend
       project: web-app
       environment: staging
   ```

3. **Security**
   - Implement network policies
   - Use RBAC
   - Regular access audits

4. **Resource Management**
   - Set resource quotas
   - Monitor resource usage
   - Implement limit ranges

## Common Use Cases

### 1. Development Lifecycle

```bash
# Create namespaces for each stage
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production
```

### 2. Team Workspaces

```bash
# Create team namespaces
kubectl create namespace team-frontend
kubectl create namespace team-backend
kubectl create namespace team-data
```

### 3. Feature Development

```bash
# Create feature-specific namespaces
kubectl create namespace feature-auth
kubectl create namespace feature-payment
```

### 4. Customer Isolation

```bash
# Create customer-specific namespaces
kubectl create namespace customer-a
kubectl create namespace customer-b
```

## Troubleshooting

1. **Common Issues**
   - Namespace deletion stuck
   - Resource quota exceeded
   - Permission denied
   - Service discovery issues

2. **Debugging Commands**

```bash
# Check namespace status
kubectl describe namespace problem-namespace

# Check namespace events
kubectl get events -n problem-namespace

# Check namespace resources
kubectl get all -n problem-namespace

# Check namespace quotas
kubectl describe quota -n problem-namespace
```

3. **Best Practices**
   - Regular namespace cleanup
   - Monitor resource usage
   - Implement proper logging
   - Regular access reviews

## Limitations

1. **Cannot Nest Namespaces**
   - Namespaces are flat
   - No hierarchy support

2. **Some Resources are Non-Namespaced**
   - Nodes
   - PersistentVolumes
   - ClusterRoles
   - Namespaces themselves

3. **Cross-Namespace Limitations**
   - Service discovery complexity
   - Resource sharing restrictions
   - Network policy considerations
