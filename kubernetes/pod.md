# Comprehensive Kubernetes Pod Guide

## Overview

A Pod is the smallest deployable unit in Kubernetes that represents a single instance of a running process in a cluster.

### Key Concepts
- **Pod**: A group of one or more containers sharing storage, network, and namespace
- **Pod ID**: Unique identifier assigned by Kubernetes
- **Pod IP**: Unique IP address within the cluster network
- **Container Runtime**: Software responsible for running containers (e.g., containerd, CRI-O)

## Pod Lifecycle States

1. **Pending**
   - Pod accepted by cluster but containers not created
   - Waiting for scheduling or image downloads

2. **Running**
   - Pod bound to node
   - All containers created
   - At least one container running or starting

3. **Succeeded**
   - All containers terminated successfully
   - Will not be restarted

4. **Failed**
   - All containers terminated
   - At least one container failed with non-zero exit code

5. **Unknown**
   - Pod state cannot be obtained
   - Usually communication error with node

6. **CrashLoopBackOff**
   - Container fails to start repeatedly
   - Kubernetes will retry with exponential back-off

7. **ImagePullBackOff**
   - Container image cannot be pulled
   - Usually due to invalid image or registry issues

## Container States

### Waiting
- Container waiting for another operation
- Common reasons: pulling image, starting dependencies

### Running
- Container executing without issues
- Process started and running

### Terminated
- Container completed execution
- Process exited or failed

## Multi-Container Pods

### Design Patterns

1. **Sidecar Pattern**
   - Enhances main container functionality
   - Example: log shippers, monitoring agents

2. **Ambassador Pattern**
   - Proxy for accessing external services
   - Example: Redis ambassador connecting to sharded instances

3. **Adapter Pattern**
   - Standardizes output of main container
   - Example: Metric exporters for monitoring

### Shared Resources
- Network namespace (localhost)
- Storage volumes
- IPC namespace
- PID namespace (optional)

### Configuration Options
- Individual resource limits/requests
- Separate security contexts
- Different probe configurations
  - Readiness probes
  - Liveness probes
  - Startup probes
- Container-specific environment variables

## Init Containers

### Purpose
- Perform initialization tasks
- Run sequentially before app containers
- Must complete successfully for pod to start

### Common Use Cases
1. Schema setup and migrations
2. Configuration templating
3. Service dependency checks
4. Security setup (certificates, keys)
5. Data download and preparation

### Key Features
- Always run to completion
- Each must complete before next starts
- Failures cause pod restart
- Support different resource requests/limits

## Pod Management Commands

### Basic Operations

```bash
# Create Pod
kubectl run nginx --image=nginx

# Create Pod with Resource Limits
kubectl run nginx --image=nginx \
  --requests='cpu=100m,memory=256Mi' \
  --limits='cpu=200m,memory=512Mi'

# Create Pod with Custom Labels
kubectl run nginx --image=nginx \
  --labels="app=web,env=prod,tier=frontend"

# Create Pod with Environment Variables
kubectl run nginx --image=nginx \
  --env="DB_HOST=mysql" \
  --env="DB_PORT=3306"
```

### Advanced Operations

```bash
# Create Pod with Volume Mount
kubectl run nginx --image=nginx \
  --dry-run=client -o yaml | \
  kubectl set volume -f - \
  --add --name=data --mount-path=/data \
  --type=emptyDir -o yaml > pod.yaml

# Create Pod with Init Container
kubectl run nginx --image=nginx \
  --dry-run=client -o yaml | \
  kubectl set init-container -f - \
  --name=init --image=busybox \
  -- /bin/sh -c "echo 'init complete'" \
  -o yaml > pod.yaml

# Apply Pod Configuration
kubectl apply -f pod.yaml

# Apply with Prune (Alpha Feature)
kubectl apply --prune -f manifest.yaml \
  -l app=nginx --prune-allowlist=core/v1/Pod
```

### Debugging Commands

```bash
# Get Pod Logs
kubectl logs pod-name [-c container-name]

# Execute Command in Pod
kubectl exec pod-name [-c container-name] -- command

# Port Forward to Pod
kubectl port-forward pod-name local-port:pod-port

# Describe Pod Details
kubectl describe pod pod-name

# Get Pod YAML
kubectl get pod pod-name -o yaml
```

## Best Practices

### Security
1. Use non-root users in containers
2. Enable read-only root filesystem
3. Set security contexts appropriately
4. Use network policies to restrict traffic

### Resource Management
1. Set appropriate resource requests/limits
2. Use horizontal pod autoscaling
3. Configure pod disruption budgets
4. Implement proper pod affinity/anti-affinity

### Monitoring
1. Configure appropriate probes
2. Set up logging properly
3. Monitor pod metrics
4. Use pod topology spread constraints

### High Availability
1. Use pod anti-affinity for spread
2. Implement proper readiness/liveness probes
3. Set appropriate restart policies
4. Use pod disruption budgets

## Common Issues and Solutions

### ImagePullBackOff
- Verify image name and tag
- Check registry credentials
- Ensure network connectivity

### CrashLoopBackOff
- Check container logs
- Verify command and args
- Check resource constraints

### Pending State
- Check node resources
- Verify node selectors
- Check PVC availability

### OOMKilled
- Increase memory limits
- Check memory leaks
- Monitor memory usage

## References
- [Kubernetes Pods Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle Guide](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Multi-Container Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)