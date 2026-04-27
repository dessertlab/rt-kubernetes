# Test Pods

Utility pods for running benchmarks, perform trials and retrieve the experiments results. Both pods include the same persistent storage used to save experiment results.

## Available Pods

### [`test-pod.yaml`](test-pod.yaml)

This pod is useful to send requests through `curl` and `hey` tools to the controllers-managed applications. However, the main goal of this pod is to mount the experiments result storage and offering easy access to it.

**Features**:
- Based on `curlimages/curl:latest`
- Automatically downloads and installs [hey](https://github.com/rakyll/hey) load generator
- Mounts persistent volume at `/experiments`
- Node-selected to specific worker node

**Usage**:
```bash
# Deploy the pod
kubectl apply -f test-pod.yaml

# Access the pod
kubectl exec -it test-pod -n default -- sh

# Run load test iside the pod
# Use curl for single requests
# Use hey for load testing
curl -v http://your-service-url
hey -z 60s -c 10 http://your-service-url

# Remove the pod after use
kubectl delete -f test-pod.yaml
```

### [`benchmark-pod.yaml`](benchmark-pod.yaml)

Generic benchmark pod based on Debian for. It can be used to manually reproduce the [vSwarm expeirments](../vSwarm-benchmarks). It mounts the persistent storage to store experimental results.

**Features**:
- Based on `debian:latest`
- Blank slate for installing custom benchmarking tools
- Mounts persistent volume at `/experiments`
- Node-selected to specific worker node

**Usage**:
```bash
# Deploy the pod
kubectl apply -f benchmark-pod.yaml

# Access the pod
kubectl exec -it benchmark-pod -- bash

# Remove the pod after use
kubectl delete -f benchmark-pod.yaml
```

Follow the vSwarm benchmark docs for instructions on how to manually try the benchamarks: https://github.com/vhive-serverless/vSwarm.

The `invoker` tool, compatible with the given pod can be found here, in this repository: [`invoker`](../vSwarm-benchmarks/invoker).

## Configuration

Both pods share common configuration:

**Node Selector**:
```yaml
nodeSelector:
  kubernetes.io/hostname: dessertw4  # Update to your node
```

**Resources**:
- CPU: 250m request, 500m limit
- Memory: 64Mi request, 128Mi limit

**Persistent Storage**:
```yaml
volumeMounts:
  - name: experiments-results
    mountPath: /experiments
```

Requires the `experiments-results` PVC (see [k8s-results-store](../k8s-results-store/)).

## Prerequisites

- `experiments-results` PVC must exist (see [k8s-results-store](../k8s-results-store/))
- Target node must exist in the cluster
- Sufficient resources available on the target node
