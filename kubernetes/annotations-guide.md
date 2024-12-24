# Kubernetes Annotations Guide

## Overview

Annotations are key-value pairs that attach arbitrary non-identifying metadata to objects. Unlike labels, annotations are not used to identify and select objects, but rather to store additional information for:

- Build/release/image information
- Logging/monitoring/analytics/audit repositories
- Tool information from client libraries or tools
- Rollout state and timestamp information
- Integration with third-party tools and services

## Annotation Structure

### Basic Format

```yaml
metadata:
  annotations:
    key: value
    example.com/git-hash: "abc123"
    kubectl.kubernetes.io/last-applied-configuration: '{"apiVersion":"v1"...}'
```

### Valid Annotation Keys

- Prefix (optional): Must be a DNS subdomain, less than 253 characters
- Name: Must be less than 63 characters
- Can contain: alphanumeric characters, dashes (-), underscores (_), dots (.)

## Common Annotation Patterns

### Build Information

```yaml
metadata:
  annotations:
    kubernetes.io/change-cause: "Image updated to v2.0.1"
    git.example.com/commit: "abc123def456"
    git.example.com/branch: "feature/new-ui"
    jenkins.io/build-number: "42"
```

### Deployment Information

```yaml
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubernetes.io/timestamp: "2024-01-01T12:00:00Z"
    kubernetes.io/rollout-status: "completed"
```

### Service Configuration

```yaml
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:..."
    nginx.ingress.kubernetes.io/rewrite-target: "/"
```

### Pod Configuration

```yaml
metadata:
  annotations:
    pod.beta.kubernetes.io/init-containers: '[...]'
    pod.alpha.kubernetes.io/initialized: "true"
    seccomp.security.alpha.kubernetes.io/pod: "docker/default"
```

## Common Use Cases

### 1. CI/CD Information

```yaml
metadata:
  annotations:
    pipeline.example.com/job-name: "main-build"
    pipeline.example.com/build-url: "https://ci.example.com/build/123"
    deploy.example.com/deployer: "jenkins"
    deploy.example.com/timestamp: "2024-01-01T12:00:00Z"
```

### 2. Configuration Management

```yaml
metadata:
  annotations:
    config.example.com/checksum: "abc123def456"
    config.example.com/version: "v1.2.3"
    config.example.com/last-modified: "2024-01-01T12:00:00Z"
    config.example.com/last-modified-by: "user@example.com"
```

### 3. Ingress Controllers

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
```

### 4. Prometheus Monitoring

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
```

## Working with Annotations

### Command Line Operations

```bash
# Add annotation
kubectl annotate pods my-pod example.com/annotation-key="value"

# Update annotation
kubectl annotate pods my-pod example.com/annotation-key="new-value" --overwrite

# Remove annotation
kubectl annotate pods my-pod example.com/annotation-key-

# View annotations
kubectl get pod my-pod -o jsonpath='{.metadata.annotations}'
```

### YAML Examples

#### Pod with Annotations

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotated-pod
  annotations:
    example.com/git-hash: "abc123"
    example.com/build-date: "2024-01-01"
    example.com/commit-msg: "Initial commit"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
```

#### Deployment with Annotations

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: annotated-deployment
  annotations:
    kubernetes.io/change-cause: "Updated to new image version"
    deployment.kubernetes.io/revision: "2"
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
```

## Best Practices

1. **Structured Data**
   - Use JSON or YAML strings for complex values
   - Maintain consistent data formats
   - Document annotation schemas

2. **Naming Conventions**
   - Use DNS-style prefixes for custom annotations
   - Keep names descriptive and meaningful
   - Use consistent casing and separators

3. **Version Control**
   - Include version information in annotations
   - Track changes through annotations
   - Maintain history of modifications

4. **Security Considerations**
   - Don't store sensitive data in annotations
   - Use appropriate access controls
   - Regular audit of annotation usage

## System Annotations

Kubernetes automatically adds certain annotations:

```yaml
metadata:
  annotations:
    kubernetes.io/created-by: "..."
    kubernetes.io/change-cause: "..."
    deployment.kubernetes.io/desired-replicas: "..."
    deployment.kubernetes.io/max-replicas: "..."
```

## Troubleshooting

1. **Common Issues**
   - Invalid annotation format
   - Missing required annotations
   - Conflicting annotations
   - Size limitations

2. **Validation**
   - Check annotation syntax
   - Verify prefix requirements
   - Validate value formats

3. **Debugging**

   ```bash
   # Get detailed object information
   kubectl describe pod my-pod
   
   # Export annotations to JSON
   kubectl get pod my-pod -o jsonpath='{.metadata.annotations}' > annotations.json
   ```

## Differences from Labels

1. **Purpose**
   - Labels: Identify and select objects
   - Annotations: Store metadata and configuration

2. **Usage**
   - Labels: Used by selector queries
   - Annotations: Used by external tools and libraries

3. **Size**
   - Labels: Limited to 63 characters
   - Annotations: Can store larger data

4. **Content**
   - Labels: Simple key-value pairs
   - Annotations: Can contain complex structured data
