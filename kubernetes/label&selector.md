# Kubernetes Labels Guide

## Overview

Labels are key-value pairs that can be attached to Kubernetes objects for organization and selection. They enable users to map their organizational structures onto system objects in a loosely coupled fashion.

## Label Syntax

### Basic Format

```yaml
metadata:
  labels:
    key: value
    app: nginx
    environment: production
    tier: frontend
```

### Valid Label Values

- Must be 63 characters or less
- Can begin and end with alphanumeric characters ([a-z0-9A-Z])
- Can contain dashes (-), underscores (_), dots (.), and alphanumerics

### Prefixed Labels

```yaml
metadata:
  labels:
    example.com/team: frontend
    kubernetes.io/created-by: controller
    cloud.google.com/gke-nodepool: main-pool
```

## Common Label Patterns

### Recommended Labels

```yaml
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/created-by: controller-manager
```

### Environment Labels

```yaml
metadata:
  labels:
    environment: production
    env: prod
    stage: staging
    tier: frontend
```

### Release Labels

```yaml
metadata:
  labels:
    release: stable
    version: v1.0.0
    track: daily
    channel: alpha
```

## Label Selectors

### Equality-Based Selectors

```bash
# Select pods with environment=production
kubectl get pods -l environment=production

# Select pods that don't have environment=production
kubectl get pods -l environment!=production

# Multiple conditions
kubectl get pods -l app=nginx,environment=production
```

### Set-Based Selectors

```bash
# Select pods where environment is either production or staging
kubectl get pods -l 'environment in (production,staging)'

# Select pods where environment is not production or staging
kubectl get pods -l 'environment notin (production,staging)'

# Select pods that have the 'version' label
kubectl get pods -l 'version'

# Select pods that don't have the 'version' label
kubectl get pods -l '!version'
```

## Label Operations

### Adding Labels

```bash
# Add label to pod
kubectl label pods my-pod environment=production

# Add label to all pods
kubectl label pods --all environment=production

# Add label to node
kubectl label node my-node disk=ssd
```

### Updating Labels

```bash
# Override existing label
kubectl label --overwrite pods my-pod environment=staging

# Update label using resource file
kubectl apply -f updated-labels.yaml
```

### Removing Labels

```bash
# Remove specific label
kubectl label pods my-pod environment-

# Remove label from all pods
kubectl label pods --all environment-
```

## Label Selectors in YAML

### Pod Selector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx
```

### Service Selector

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
    environment: production
  ports:
  - port: 80
```

### Deployment Selector

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
```

## Best Practices

1. **Meaningful Labels**
   - Use descriptive keys and values
   - Follow consistent naming conventions
   - Use prefixes for organizational labels

2. **Label Hierarchy**

   ```yaml
   metadata:
     labels:
       organization: company-name
       team: team-name
       project: project-name
       component: component-name
   ```

3. **Standard Label Sets**
   - Use Kubernetes recommended labels
   - Include version information
   - Label both apps and infrastructure

4. **Label Management**
   - Document label schema
   - Use automation for consistency
   - Regular label audits

## Common Use Cases

1. **Resource Organization**

   ```yaml
   metadata:
     labels:
       app: myapp
       component: frontend
       environment: production
   ```

2. **Deployment Strategies**

   ```yaml
   metadata:
     labels:
       version: v1.0.0
       track: stable
       deployment: blue
   ```

3. **Cost Allocation**

   ```yaml
   metadata:
     labels:
       cost-center: team-123
       project: project-abc
       billing: department-xyz
   ```

4. **Security Policies**

   ```yaml
   metadata:
     labels:
       security-zone: restricted
       compliance: pci-dss
       data-sensitivity: high
   ```

## Label Query Examples

```bash
# Complex label selections
kubectl get pods -l 'environment in (production,staging),tier notin (frontend)'

# Multiple conditions with set-based requirements
kubectl get pods -l 'app=nginx,environment in (production,staging)'

# Node selection for pod scheduling
kubectl get nodes -l 'kubernetes.io/role=node,disk=ssd'

# Service endpoint selection
kubectl get endpoints -l 'app=nginx,environment=production'
```

## Troubleshooting

1. **Label Conflicts**
   - Check for duplicate labels
   - Verify label selector syntax
   - Ensure label values are valid

2. **Selector Issues**
   - Verify label existence
   - Check selector syntax
   - Validate matchLabels vs matchExpressions

3. **Common Problems**
   - Missing required labels
   - Incorrect label values
   - Selector mismatch
