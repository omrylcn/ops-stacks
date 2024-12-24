# Kubernetes Deployments Guide

## Overview

Deployments provide declarative updates for Pods and ReplicaSets. They are the recommended way to manage stateless applications in Kubernetes, offering features like rolling updates, rollbacks, and scaling capabilities.

## Core Concepts

### 1. Pod Management

Deployments manage Pods through ReplicaSets, ensuring:

- Desired number of Pods are running
- Updates are rolled out systematically
- Application state matches the declared configuration

### 2. ReplicaSet Relationship

- Deployments create and manage ReplicaSets
- Each update creates a new ReplicaSet
- Old ReplicaSets are maintained for rollback purposes
- ReplicaSets ensure Pod count matches specifications

## Basic Operations

### Command Line Operations

```bash
# Create a deployment
kubectl create deployment nginx --image=nginx:1.14

# Get deployments
kubectl get deployments
kubectl get deploy

# Describe deployment
kubectl describe deployment nginx

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.15

# Check rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Delete deployment
kubectl delete deployment nginx
```

### YAML Definitions

#### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:1.14
        ports:
        - containerPort: 80
```

#### Advanced Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
  labels:
    app: web
    environment: prod
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web-app
        image: nginx:1.14
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

## Deployment Features

### 1. Update Strategies

#### RollingUpdate (Default)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

#### Recreate

```yaml
spec:
  strategy:
    type: Recreate
```

### 2. Scaling

```yaml
spec:
  replicas: 5  # Static scaling
```

```bash
# Dynamic scaling
kubectl scale deployment nginx --replicas=3
```

### 3. Rolling Updates

The process of updating Pods gradually without downtime:

1. Create new ReplicaSet with updated configuration
2. Gradually scale up new ReplicaSet
3. Gradually scale down old ReplicaSet
4. Maintain application availability throughout

### 4. Rollbacks

```bash
# View rollout history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2
```

## Best Practices

1. **Resource Management**
   - Always specify resource requests and limits
   - Use horizontal pod autoscaling when appropriate
   - Monitor resource usage

2. **Update Strategy**
   - Use RollingUpdate for zero-downtime deployments
   - Configure appropriate maxSurge and maxUnavailable
   - Implement proper health checks

3. **Labels and Selectors**
   - Use meaningful labels
   - Maintain consistent labeling scheme
   - Avoid selector conflicts

4. **Health Checks**
   - Implement readiness probes
   - Configure liveness probes
   - Set appropriate timing parameters

## Common Use Cases

### 1. Web Applications

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-frontend
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```

### 2. API Services

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: api
        image: api-server:1.0
        ports:
        - containerPort: 8080
```

## Troubleshooting

1. **Common Issues**
   - Failed rollouts
   - Resource constraints
   - Image pull errors
   - Probe failures

2. **Debugging Commands**

```bash
# Check deployment status
kubectl describe deployment problematic-deployment

# Check pods status
kubectl get pods -l app=problematic-app

# Check rollout status
kubectl rollout status deployment/problematic-deployment

# View deployment events
kubectl get events --field-selector involvedObject.kind=Deployment
```

3. **Best Practices**
   - Monitor deployment metrics
   - Set up proper logging
   - Implement appropriate alerts
   - Maintain deployment documentation

## Limitations

1. **State Management**
   - Not suitable for stateful applications
   - Use StatefulSets for stateful workloads

2. **Update Constraints**
   - Cannot update selector labels
   - Template updates trigger new rollouts
   - Storage configuration changes require manual intervention

3. **Scaling Limitations**
   - Resource availability affects scaling
   - Network policies may impact scaling
   - Pod affinity rules can limit scheduling
