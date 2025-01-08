# Comprehensive Kubernetes Network Policy Guide

## Overview

Network Policies are Kubernetes resources that specify how groups of pods are allowed to communicate with each other and with external network endpoints. They act as the Kubernetes equivalent of a firewall, providing fine-grained control over pod-to-pod and external communication.

### Key Concepts

- **Network Policy**: A specification of how groups of pods are allowed to communicate
- **Ingress**: Incoming traffic to pods
- **Egress**: Outgoing traffic from pods
- **Selectors**: Rules that identify pods and namespaces affected by policies
- **CIDR Blocks**: IP address ranges for external traffic control

## Network Policy Behavior

### Default Settings

By default, pods accept traffic from any source (ingress) and can send traffic to any destination (egress). When any Network Policy selects a pod, that pod will reject all traffic not specifically allowed by a Network Policy. This behavior makes Network Policies additive, meaning:

- Multiple policies can select the same pod
- Traffic is allowed if any policy allows it
- You can't create policies that deny traffic
- Traffic is denied by default when a pod is selected by any policy

### Policy Evaluation Process

1. Pod selection through labels
2. Check if pod is subject to any Network Policies
3. Evaluate all policies selecting the pod
4. Allow traffic if any policy permits it
5. Deny all other traffic

## Policy Types and Components

### Ingress Policies

Control incoming traffic to selected pods. Common use cases include:

- Allowing traffic only from specific namespaces
- Restricting access to databases from authorized applications
- Implementing network segmentation between environments

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              environment: production
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### Egress Policies

Control outgoing traffic from selected pods. Useful for:

- Limiting external API access
- Controlling outbound traffic for compliance
- Preventing unauthorized data exfiltration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-egress
  namespace: database
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/16
            except:
              - 10.0.5.0/24
      ports:
        - protocol: TCP
          port: 5432
```

## Advanced Policy Patterns

### Zero-Trust Network Model

Implement a zero-trust architecture by denying all traffic by default and explicitly allowing required communications:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### Multi-tier Application Isolation

Secure communication between application tiers:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: frontend
      ports:
        - port: 8080
          protocol: TCP
```

### Cross-namespace Communication

Control traffic between different namespaces:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cross-namespace
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              purpose: monitoring
        - podSelector:
            matchLabels:
              role: prometheus
```

## Best Practices

### Security

1. Implement default deny policies for both ingress and egress
2. Use specific CIDR ranges instead of allowing all IPs
3. Regularly audit and review Network Policies
4. Document policy intentions and requirements

### Performance

1. Minimize the number of policies per namespace
2. Use efficient label selectors
3. Avoid overly complex policy combinations
4. Monitor policy evaluation impact

### Management

1. Use consistent labeling schemes
2. Implement change control for policy updates
3. Maintain policy documentation
4. Test policies in non-production first

## Troubleshooting

### Common Issues

1. **Policy Not Applied**
   - Verify CNI plugin supports Network Policies
   - Check label selectors match intended pods
   - Confirm policy is in correct namespace

2. **Unexpected Traffic Blocking**
   - Review all policies affecting the pod
   - Check for conflicting policies
   - Verify label selections

3. **External Traffic Issues**
   - Validate CIDR ranges
   - Check for default deny policies
   - Verify egress rules for external services

### Debugging Commands

```bash
# List Network Policies
kubectl get networkpolicy -A

# Describe Network Policy
kubectl describe networkpolicy policy-name -n namespace

# Check Pod Labels
kubectl get pod pod-name --show-labels

# Validate Policy YAML
kubectl get networkpolicy policy-name -o yaml
```

## CNI Plugin Considerations

### Supported Features

Different CNI plugins support varying Network Policy features:

1. **Calico**
   - Full Network Policy support
   - Advanced policy features
   - Good performance at scale

2. **Cilium**
   - Layer 7 policy support
   - Advanced security features
   - High performance

3. **Weave Net**
   - Basic Network Policy support
   - Simple configuration
   - Good for smaller clusters

### Implementation Requirements

1. CNI plugin must support Network Policies
2. Plugin must be properly configured
3. Cluster networking must be correctly setup
4. Node-to-node communication must be allowed

## Advanced Topics

### Policy Metrics

Monitor policy effectiveness:

1. Number of allowed/denied connections
2. Policy evaluation time
3. Policy match rates
4. Rule hit counts

### Integration with Service Mesh

Combine Network Policies with service mesh features:

1. Use Network Policies for coarse-grained control
2. Implement fine-grained control in service mesh
3. Layer security controls appropriately
4. Monitor both policy layers

## References

- [Kubernetes Network Policies Documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Network Policy Recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)
- [CNI Specification](https://github.com/containernetworking/cni/blob/master/SPEC.md)
