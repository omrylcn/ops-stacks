# Kubernetes Configuration and Commands Guide

## Kubeconfig File

The kubeconfig file is typically located at `~/.kube/config` and uses YAML format to organize:

- Clusters: Define the various clusters you can access
- Users: Define the user credentials for authentication
- Contexts: Define combinations of clusters and users
- Current-context: Specifies which context is currently active

### Config File Structure

```yaml
apiVersion: v1
kind: Config
preferences: {}

clusters:
- name: development
  cluster:
    server: https://1.2.3.4:6443
    certificate-authority-data: DATA+OMITTED
    # OR certificate-authority: /path/to/ca.crt
    # OR insecure-skip-tls-verify: true

users:
- name: developer
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
    # OR client-certificate: /path/to/cert.crt
    # OR client-key: /path/to/key.key
    # OR token: bearer_token
    # OR username/password

contexts:
- name: dev-context
  context:
    cluster: development
    user: developer
    namespace: dev-namespace # optional

current-context: dev-context
```

### Key Components

1. **Clusters Section**
   - `name`: Unique identifier for the cluster
   - `server`: API server endpoint
   - `certificate-authority-data`: Base64 encoded CA cert
   - `insecure-skip-tls-verify`: Skip TLS verification (not recommended for production)

2. **Users Section**
   - `name`: Username for authentication
   - Authentication methods:
     - Client certificates
     - Client keys
     - Tokens
     - Basic auth (username/password)
     - OIDC tokens
     - AWS IAM authenticator

3. **Contexts Section**
   - `name`: Context identifier
   - `cluster`: Reference to cluster name
   - `user`: Reference to user name
   - `namespace`: Default namespace (optional)

### Multiple Config Files

You can merge multiple kubeconfig files using:

```bash
# Set KUBECONFIG environment variable
export KUBECONFIG=file1:file2:file3

# Merge configs into one file
kubectl config view --flatten > ~/.kube/merged_config
```

## Context Management

A context in Kubernetes is a set of access parameters including:

- Cluster: The Kubernetes cluster to connect to
- User: The user credentials to use
- Namespace: The default namespace for commands

### Context Commands

```bash
# List all contexts
kubectl config get-contexts

# Display current context
kubectl config current-context

# Switch to a different context
kubectl config use-context CONTEXT_NAME

# Set default namespace for current context
kubectl config set-context --current --namespace=NAMESPACE

# Delete a context
kubectl config delete-context CONTEXT_NAME

# Rename a context
kubectl config rename-context OLD_NAME NEW_NAME

# Show merged kubeconfig settings
kubectl config view

# Show specific config file settings
kubectl config view --kubeconfig=FILENAME

# Display cluster endpoints
kubectl cluster-info
```

## Core Commands

### Cluster Information

```bash
# Display cluster information
kubectl cluster-info

# Show detailed cluster dump
kubectl cluster-info dump

# View version information
kubectl version

# Check API resources
kubectl api-resources
```

### Resource Management

#### Get Resources

```bash
# List all pods
kubectl get pods

# List all services
kubectl get services

# List multiple resource types
kubectl get pods,services

# List all resources in all namespaces
kubectl get all --all-namespaces

# List with custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# List with JSON path
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Watch resource changes
kubectl get pods --watch

# List resources with labels
kubectl get pods -l app=nginx
```

#### Describe Resources

```bash
# Detailed information about a specific pod
kubectl describe pod POD_NAME

# Describe all pods matching a label
kubectl describe pods -l app=nginx

# Describe multiple resource types
kubectl describe pods,services

# Describe resources in different namespace
kubectl describe pod POD_NAME -n NAMESPACE
```

#### Resource Documentation

```bash
# Get documentation for pod
kubectl explain pod

# Get documentation for specific field
kubectl explain pod.spec.containers

# Get documentation with recursive flag
kubectl explain pod --recursive
```

### Pod Operations

#### Logs

```bash
# Get pod logs
kubectl logs POD_NAME

# Get logs for specific container in pod
kubectl logs POD_NAME -c CONTAINER_NAME

# Follow logs in real-time
kubectl logs -f POD_NAME

# Get logs since specific time
kubectl logs POD_NAME --since=1h

# Get specific number of lines
kubectl logs POD_NAME --tail=100

# Get logs from all containers in pod
kubectl logs POD_NAME --all-containers=true
```

#### Exec Commands

```bash
# Execute command in pod
kubectl exec POD_NAME -- COMMAND

# Interactive shell
kubectl exec -it POD_NAME -- /bin/bash

# Execute in specific container
kubectl exec POD_NAME -c CONTAINER_NAME -- COMMAND

# Copy files to/from pod
kubectl cp POD_NAME:SOURCE_PATH DESTINATION_PATH
```

### Resource Modification

#### Edit Resources

```bash
# Edit pod
kubectl edit pod POD_NAME

# Edit service
kubectl edit service SERVICE_NAME

# Edit with specific editor
KUBE_EDITOR="nano" kubectl edit pod POD_NAME

# Edit in specific namespace
kubectl edit pod POD_NAME -n NAMESPACE
```

#### Delete Resources

```bash
# Delete pod
kubectl delete pod POD_NAME

# Delete all pods matching label
kubectl delete pods -l app=nginx

# Delete multiple resource types
kubectl delete pods,services -l app=nginx

# Force delete pod
kubectl delete pod POD_NAME --force --grace-period=0

# Delete all resources in namespace
kubectl delete all --all -n NAMESPACE
```

## Advanced Commands

### Configuration Management

```bash
# Create configmap from file
kubectl create configmap NAME --from-file=PATH

# Create secret
kubectl create secret generic NAME --from-literal=key=value

# Set image
kubectl set image deployment/DEPLOYMENT CONTAINER=IMAGE
```

### Resource Scaling

```bash
# Scale deployment
kubectl scale deployment DEPLOYMENT --replicas=3

# Autoscale deployment
kubectl autoscale deployment DEPLOYMENT --min=2 --max=5 --cpu-percent=80
```

### Rolling Updates

```bash
# Rollout status
kubectl rollout status deployment/DEPLOYMENT

# Rollout history
kubectl rollout history deployment/DEPLOYMENT

# Rollback to previous version
kubectl rollout undo deployment/DEPLOYMENT
```

## Best Practices

1. **Context Management**
   - Regularly verify current context
   - Use meaningful context names
   - Set appropriate default namespaces

2. **Resource Operations**
   - Use labels for organization
   - Always specify namespace when needed
   - Use resource limits and requests

3. **Logging and Debugging**
   - Use appropriate log levels
   - Set reasonable log retention
   - Use describe for troubleshooting

4. **Security**
   - Use RBAC appropriately
   - Regularly rotate credentials
   - Follow principle of least privilege

## Config File Management Commands

```bash
# View current config
kubectl config view

# View specific config file
kubectl config view --kubeconfig=specific-config

# Get specific value from config
kubectl config view -o jsonpath='{.current-context}'

# Set a cluster entry
kubectl config set-cluster my-cluster \
    --server=https://1.2.3.4:6443 \
    --certificate-authority=ca.crt

# Set credentials
kubectl config set-credentials my-user \
    --client-certificate=client.crt \
    --client-key=client.key

# Set a context
kubectl config set-context my-context \
    --cluster=my-cluster \
    --user=my-user \
    --namespace=my-namespace

# Modify context settings
kubectl config set contexts.my-context.namespace new-namespace

# Unset a value
kubectl config unset users.my-user

# Delete cluster entry
kubectl config delete-cluster my-cluster

# Delete user entry
kubectl config delete-user my-user
```

## Common Options

- `-n, --namespace`: Specify namespace
- `-f`: Specify filename
- `-l`: Label selector
- `-o`: Output format
- `--all-namespaces`: All namespaces
- `--force`: Force operation
- `--grace-period`: Grace period for operation
- `--dry-run`: Test run without actual execution

## Environment Variables

```bash
# Set namespace for current context
export KUBE_NAMESPACE=my-namespace

# Set context
export KUBECONFIG=/path/to/kubeconfig

# Set editor
export KUBE_EDITOR=vim
```

## Useful Aliases

```bash
# Common aliases for kubectl commands
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias ka='kubectl apply -f'
alias kdel='kubectl delete'
```

