# Kubernetes Components 102


## I. SECURITY AND ACCESS MANAGEMENT

### A. Role Based Access Control (RBAC)

RBAC allows us to manage access permissions for different components and users in the MLOps ecosystem in a granular way.

```yaml
# RBAC configuration for Model Training Team
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: model-trainer
rules:
- apiGroups: [""]
  resources: ["pods", "jobs"]
  verbs: ["create", "get", "list"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["create", "delete"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: model-trainer-binding
subjects:
- kind: ServiceAccount
  name: training-account
roleRef:
  kind: Role
  name: model-trainer
  apiGroup: rbac.authorization.k8s.io
```

In this configuration, the model training team is given only the minimum necessary permissions. For example, they can create and monitor training jobs but cannot delete them.

### B. Service Accounts

Service Accounts enable ML pipelines and applications to interact securely with the Kubernetes API.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: model-serving
  annotations:
    kubernetes.io/enforce-mountable-secrets: "true"
---
# Creating custom secret for Service Account
apiVersion: v1
kind: Secret
metadata:
  name: model-serving-token
  annotations:
    kubernetes.io/service-account.name: model-serving
type: kubernetes.io/service-account-token
```

## II. STORAGE MANAGEMENT

### A. PersistentVolume and PersistentVolumeClaim

Persistent storage management is critical for ML workloads, especially for large datasets and model files.

```yaml
# PV for high-performance storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: training-data-pv
spec:
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  nfs:
    server: nfs-server.example.com
    path: "/training-data"
---
# PVC for training jobs
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: training-data-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Gi
  storageClassName: high-performance
```

### B. StorageClass

Storage solutions with different performance characteristics for different ML workloads:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: high-performance
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "50"
  fsType: ext4
```

## III. WORKLOAD MANAGEMENT

### A. Job and CronJob

Batch job management for model training and retraining operations:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: model-retraining
spec:
  schedule: "0 2 * * *"  # Every night at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: training
            image: ml-training:v1
            resources:
              requests:
                nvidia.com/gpu: 1
            volumeMounts:
            - name: training-data
              mountPath: /data
            - name: model-output
              mountPath: /models
          volumes:
          - name: training-data
            persistentVolumeClaim:
              claimName: training-data-claim
          - name: model-output
            persistentVolumeClaim:
              claimName: model-storage-claim
```

### B. StatefulSet

Management of worker nodes for distributed training:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: distributed-training
spec:
  serviceName: training-cluster
  replicas: 4
  template:
    spec:
      containers:
      - name: training-worker
        image: distributed-training:v1
        env:
        - name: POD_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.ordinal
```

## IV. RESOURCE MANAGEMENT AND SCHEDULING

### A. Node/Pod Affinity

Scheduling rules for efficient use of GPU resources:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-training
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: nvidia.com/gpu
            operator: Exists
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: training-job
              operator: In
              values:
              - gpu-intensive
          topologyKey: kubernetes.io/hostname
```

### B. Taint and Toleration

Node isolation for ML workloads requiring specialized hardware:

```yaml
# Taint for GPU nodes
kubectl taint nodes gpu-node dedicated=gpu:NoSchedule

# Toleration for training pod
apiVersion: v1
kind: Pod
metadata:
  name: gpu-training
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

## V. CONFIGURATION AND SECRET MANAGEMENT

### A. ConfigMap

Model parameters and environment variables:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: model-params
data:
  learning_rate: "0.001"
  batch_size: "32"
  epochs: "100"
  model_architecture: |
    {
      "layers": [
        {"type": "Dense", "units": 128},
        {"type": "Dropout", "rate": 0.5}
      ]
    }
```

### B. Secret

API key and credentials management:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ml-credentials
type: Opaque
data:
  model_registry_key: <base64-encoded>
  monitoring_api_key: <base64-encoded>
```

## VI. SERVICE AND NETWORK MANAGEMENT

### A. Ingress

Management of model serving endpoints:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: model-serving
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: ml-api.example.com
    http:
      paths:
      - path: /v1/predict
        pathType: Prefix
        backend:
          service:
            name: model-serving
            port:
              number: 8080
```

## BEST PRACTICES AND RECOMMENDATIONS

### 1. Resource Management

- Define appropriate resource limits for each ML workload
- Optimize node affinity rules for efficient use of GPU resources

### 2. Security

- Design RBAC rules according to the principle of least privilege
- Store all credentials in Secrets
- Restrict pod-to-pod communication with network policies

### 3. Monitoring

- Collect ML metrics with Prometheus
- Define custom resource metrics
- Create alert rules

### 4. Storage

- Choose appropriate StorageClass for I/O performance-critical workloads
- Create backup and disaster recovery plans