# kubbectl Cheat Sheet

## Cluster Management

```bash
# Basic Cluster Commands
kubectl cluster-info                              # Display cluster endpoint information
kubectl version                                   # Show Kubernetes version
kubectl config view                              # Show kubeconfig settings
kubectl api-resources                            # List API resources
kubectl api-versions                             # List API versions
kubectl get all --all-namespaces                 # List everything
```

## Pod Operations

```bash
# Basic Pod Commands (Shortcode = po)
kubectl get pods                                 # List pods
kubectl get pods -o wide                         # List pods with more details
kubectl describe pod <pod-name>                  # Show pod details
kubectl delete pod <pod-name>                    # Delete pod
kubectl create pod <pod-name>                    # Create pod
kubectl exec -it <pod-name> -- /bin/bash        # Get shell inside pod
kubectl exec <pod-name> -c <container> <command> # Execute command in pod/container

# Pod Resource Usage
kubectl top pod                                  # Show pod resource usage
kubectl annotate pod <pod-name> <annotation>     # Add/update pod annotations
kubectl label pod <pod-name> key=value          # Add/update pod labels
```

## Logging Operations

```bash
# Basic Log Commands
kubectl logs <pod_name>                          # Print pod logs
kubectl logs -f <pod_name>                       # Stream pod logs
kubectl logs --since=1h <pod_name>               # Show logs for last hour
kubectl logs --tail=20 <pod_name>                # Show last 20 lines of logs
kubectl logs -c <container_name> <pod_name>      # Show container logs
kubectl logs --previous <pod_name>               # Show logs from previous container
```

## Namespace Operations (Shortcode = ns)

```bash
# Namespace Management
kubectl get namespace                            # List namespaces
kubectl create namespace <name>                  # Create namespace
kubectl delete namespace <name>                  # Delete namespace
kubectl describe namespace <name>                # Describe namespace
kubectl config set-context --current --namespace=<name> # Switch namespace
kubectl top namespace <name>                     # Show namespace resource usage
```

## Deployment Operations (Shortcode = deploy)

```bash
# Deployment Management
kubectl get deployments                          # List deployments
kubectl describe deployment <name>               # Show deployment details
kubectl create deployment <name> --image=<image> # Create deployment
kubectl delete deployment <name>                 # Delete deployment
kubectl rollout status deployment <name>         # Check rollout status
kubectl scale deployment <name> --replicas=3     # Scale deployment
kubectl rollout undo deployment <name>           # Rollback deployment
```

## Service Operations (Shortcode = svc)

```bash
# Service Management
kubectl get services                             # List services
kubectl describe service <name>                  # Show service details
kubectl expose deployment <name> --port=80       # Create service
kubectl edit service <name>                      # Edit service
kubectl delete service <name>                    # Delete service
```

## Node Operations (Shortcode = no)

```bash
# Node Management
kubectl get nodes                                # List nodes
kubectl describe node <node_name>                # Show node details
kubectl top node                                 # Show node resource usage
kubectl cordon node <node_name>                  # Mark node as unschedulable
kubectl uncordon node <node_name>                # Mark node as schedulable
kubectl drain node <node_name>                   # Drain node for maintenance
kubectl taint node <node_name>                   # Update node taints
```

## DaemonSet Operations (Shortcode = ds)

```bash
# DaemonSet Management
kubectl get daemonset                            # List daemonsets
kubectl describe ds <name>                       # Show daemonset details
kubectl edit daemonset <name>                    # Edit daemonset
kubectl delete daemonset <name>                  # Delete daemonset
kubectl rollout daemonset                        # Manage daemonset rollout
```

## ConfigMap and Secret Operations

```bash
# ConfigMap & Secret Management
kubectl create configmap <name> --from-file=<path> # Create configmap
kubectl get configmaps                           # List configmaps
kubectl create secret generic <name>             # Create secret
kubectl get secrets                              # List secrets
kubectl describe secrets                         # Show secret details
```

## Manifest File Operations

```bash
# YAML/Manifest Management
kubectl apply -f <file.yaml>                     # Apply configuration
kubectl create -f <file.yaml>                    # Create from file
kubectl create -f ./dir                          # Create from directory
kubectl delete -f <file.yaml>                    # Delete resource
```

## Additional Common Commands

```bash
# Event Management (Shortcode = ev)
kubectl get events                               # List events
kubectl get events --field-selector type=Warning # List warnings only

# Service Account Operations (Shortcode = sa)
kubectl get serviceaccounts                      # List service accounts
kubectl describe serviceaccounts                 # Show service account details

# StatefulSet Operations (Shortcode = sts)
kubectl get statefulset                          # List statefulsets
kubectl delete statefulset/<name> --cascade=false # Delete statefulset only

# Common Options
-o wide                                          # Output with more details
-n <namespace>                                   # Specify namespace
-f <filename>                                    # Specify filename
-l key=value                                     # Label selector
```

## ReplicaSet Operations (Shortcode = rs)

```bash
# ReplicaSet Management
kubectl get replicasets                          # List replicasets
kubectl describe replicasets <name>              # Show replicaset details
kubectl scale --replicas=[x] replicaset <name>   # Scale replicaset
```

## Replication Controller Operations (Shortcode = rc)

```bash
kubectl get rc                                   # List replication controllers
kubectl get rc --namespace="<namespace>"         # List RC in namespace
```

## Event Filtering Operations

```bash
# Advanced Event Filtering
kubectl get events --field-selector involvedObject.kind!=Pod  # Exclude Pod events
kubectl get events --field-selector \
  involvedObject.kind=Node,involvedObject.name=<node_name>    # Node specific events
```

## Resource Usage Commands

```bash
# Resource Monitoring
kubectl describe nodes | grep Allocated -A 5     # Resource allocation per node
kubectl top pods -n <namespace>                  # Pod resource usage in namespace
kubectl top nodes                                # Node resource usage
```

## Common Debug Commands

```bash
# Debugging
kubectl port-forward <pod_name> 8080:80          # Port forwarding
kubectl attach <pod_name> -i                     # Attach to pod
kubectl exec -it <pod_name> -- /bin/bash         # Interactive shell
kubectl run test --image=nginx --rm -it          # Run test pod and delete after

# Cluster Debug
kubectl get componentstatuses                    # Check component status
kubectl cluster-info dump                        # Dump cluster state
```

## Label and Annotation Operations

```bash
# Label Management
kubectl label pods <pod_name> key=value          # Add/update label
kubectl label pods <pod_name> key-               # Remove label
kubectl get pods --show-labels                   # Show all labels

# Annotation Management
kubectl annotate pods <pod_name> key=value       # Add annotation
kubectl annotate pods <pod_name> key-            # Remove annotation
```

## Format Output Options

```bash
# Output Formatting
kubectl get pods -o yaml                         # Output in YAML format
kubectl get pods -o json                         # Output in JSON format
kubectl get pods -o jsonpath='{.items[*].metadata.name}'  # Get pod names
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase  # Custom columns
```

## Watch Commands

```bash
# Real-time Monitoring
kubectl get pods -w                              # Watch pods
kubectl get nodes -w                             # Watch nodes
kubectl get deployments -w                       # Watch deployments
```

## Context Management

```bash
# Context Operations
kubectl config get-contexts                      # List contexts
kubectl config use-context <context_name>        # Switch context
kubectl config current-context                   # Show current context
kubectl config delete-context <context_name>     # Delete context
```

## Advanced Commands

```bash
# Advanced Operations
kubectl diff -f <file.yaml>                      # Show changes to be applied
kubectl patch deployment <name> -p '<patch>'     # Patch resource
kubectl replace --force -f <file.yaml>           # Force replace resource
kubectl wait --for=condition=Ready pod/<name>    # Wait for condition
```

1. Help can be obtained with the `-h` or `--help` parameter for each command
2. Log level can be adjusted with `--v` or `--verbosity`
3. Most commands can output in different formats with `-o wide`, `-o yaml` or `-o json`
4. Resources in all namespaces can be listed with `--all-namespaces` or `-A`
