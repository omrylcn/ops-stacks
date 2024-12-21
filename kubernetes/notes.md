- master and nodes
- pod
- declrative ad imperative
- yaml
- minikuv
- kubectl
- replication controller
- service
- deployment
  
### Kubernetes Components

- API Server: The central management point for the entire cluster that exposes the Kubernetes API and handles all internal/external communication
- etcd: Distributed key-value store used as Kubernetes' backing store for all cluster data, configuration and state
- Controller Manager: Runs controller processes that regulate the state of the cluster, ensuring desired state matches actual state through continuous monitoring and reconciliation
- Kube-Scheduler: Watches for newly created pods with no assigned node and selects an optimal node for them to run on based on resource requirements, constraints and policies
- Kubelet: An agent that runs on each node, ensuring containers are running in a pod and maintaining their health and lifecycle
- Kube-Proxy: Network proxy that maintains network rules on nodes, enabling Kubernetes service abstractions and load balancing
- Node: A worker machine (VM or physical) that runs containerized applications managed by the control plane
- Control Plane: The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers across the cluster
- Container Runtime: The software (like Docker, containerd, CRI-O) that is responsible for pulling the container image from a registry, unpacking the container image, and running the containers with proper isolation
- Cloud Provider API: An interface for cloud providers to integrate with Kubernetes, allowing for easy deployment and management of Kubernetes clusters in various cloud environments
- Kube-DNS: A DNS server that provides DNS resolution for services and pods in the cluster

