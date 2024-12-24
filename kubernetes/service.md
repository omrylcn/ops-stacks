# Kubernetes Services Guide

## Understanding Services

Think of a Kubernetes Service as a stable "front desk" for your application. Just as a hotel's front desk remains at the same location while different staff members work there, a Service provides a consistent way to access your Pods even as they come and go. This abstraction is crucial because Pods are ephemeral - they can be created, destroyed, or moved around, but your application still needs a reliable way to be accessed.

## Core Concepts

### Service Types

Services come in four main types, each serving different networking needs:

1. **ClusterIP (Default)**
   - Creates a virtual IP inside the cluster
   - Only accessible within the cluster
   - Perfect for internal communication between applications

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: backend-service
   spec:
     type: ClusterIP
     selector:
       app: backend
     ports:
       - port: 80         # Port the service listens on
         targetPort: 8080 # Port the application in the Pod listens on
   ```

2. **NodePort**
   - Extends ClusterIP
   - Opens a specific port on all Nodes
   - Makes service accessible from outside the cluster
   - Port range: 30000-32767

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-service
   spec:
     type: NodePort
     selector:
       app: web
     ports:
       - port: 80         # Internal cluster port
         targetPort: 8080 # Container port
         nodePort: 30080  # External port (optional, auto-assigned if not specified)
   ```

3. **LoadBalancer**
   - Extends NodePort
   - Provisions an external load balancer (in cloud environments)
   - Provides a single external IP to access the service

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-external
   spec:
     type: LoadBalancer
     selector:
       app: web
     ports:
       - port: 80
         targetPort: 8080
   ```

4. **ExternalName**
   - Maps the Service to an external DNS name
   - Used for accessing external services
   - No proxying involved

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: external-database
   spec:
     type: ExternalName
     externalName: db.example.com
   ```

## Basic Operations

### Command Line Interface

```bash
# Create a service
kubectl create service clusterip nginx --tcp=80:80

# List all services
kubectl get services
kubectl get svc

# Get detailed information about a service
kubectl describe service nginx

# Delete a service
kubectl delete service nginx

# Access a service (from within the cluster)
kubectl run temp-pod --rm -it --image=nginx -- curl service-name

# Port forward a service to your local machine
kubectl port-forward service/nginx 8080:80
```

### Service Discovery

Services enable automatic service discovery within the cluster through:

1. **Environment Variables**
   - Each Pod gets environment variables for active services
   - Format: SERVICE_NAME_SERVICE_HOST and SERVICE_NAME_SERVICE_PORT

2. **DNS**
   - Services get DNS entries automatically
   - Format: service-name.namespace.svc.cluster.local
   - This is the preferred method

## Advanced Features

### 1. Session Affinity

If you need clients to connect to the same Pod consistently:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

### 2. Multi-Port Services

When your application exposes multiple ports:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: my-app
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
```

### 3. Headless Services

For direct Pod-to-Pod communication:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: my-app
```

## Best Practices

### 1. Naming and Labels

Use meaningful, consistent names and labels for easy management:

```yaml
metadata:
  name: user-api
  labels:
    app: user-service
    tier: backend
    environment: production
```

### 2. Health Checks

Implement readiness probes so Services only route to healthy Pods:

```yaml
spec:
  template:
    spec:
      containers:
        - name: web
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

### 3. Network Policies

Secure your Services with network policies:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
```

## Troubleshooting Guide

### Common Issues and Solutions

1. **Service Not Accessible**
   - Check selector matches Pod labels
   - Verify Pods are running and ready
   - Check service ports configuration

   ```bash
   # Debug steps
   kubectl get endpoints my-service
   kubectl describe service my-service
   kubectl get pods --selector=app=my-app
   ```

2. **DNS Resolution Issues**
   - Verify CoreDNS is running
   - Check service DNS entry

   ```bash
   # Debug steps
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   kubectl run -it --rm debug --image=busybox -- nslookup my-service
   ```

3. **Load Balancer Issues**
   - Check cloud provider status
   - Verify firewall rules

   ```bash
   # Debug steps
   kubectl describe service my-loadbalancer
   kubectl get events --field-selector involvedObject.kind=Service
   ```

## Advanced Use Cases

### 1. Blue-Green Deployments

Using Services for zero-downtime deployments:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
    version: blue  # Switch between blue and green
```

### 2. Canary Deployments

Running multiple versions simultaneously:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app  # Match both versions
```

### 3. Internal Load Balancing

Using Services for internal traffic distribution:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-lb
  annotations:
    cloud.google.com/load-balancer-type: "Internal"
spec:
  type: LoadBalancer
```

## Performance Considerations

1. **Connection Handling**
   - Configure appropriate timeouts
   - Monitor connection counts
   - Consider session affinity needs

2. **Load Balancing**
   - Choose appropriate service type
   - Monitor load distribution
   - Configure health checks properly

3. **DNS Caching**
   - Be aware of DNS TTL values
   - Consider using NodeLocal DNSCache
   - Monitor DNS query patterns

Remember: Services are a fundamental building block of Kubernetes networking. Understanding them well is crucial for building reliable, scalable applications in Kubernetes.
