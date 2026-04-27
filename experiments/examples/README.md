# Application Examples

This directory contains example manifests demonstrating how to deploy critical applications using the PREEMPT-FaaS controller with RTResources custom resources and Knative services.

## Directory Structure

```
examples/
├── application-example/                        # Basic RTResource example
│   └── test-application.yaml
└── knative-application-example/                # Knative service examples
    ├── knative-test-application.yaml
    ├── apiserver-kube-manager-auditing-flux/   # Audit logs for Kube-Manager controller workflow
    └── apiserver-preempt-k8s-auditing-flux/    # Audit logs for PREEMPT-FaaS controller workflow
```

## RTResource Example

### [`application-example/test-application.yaml`](application-example/test-application.yaml)

Basic example of a critical application deployed as an RTResource.

```yaml
apiVersion: rtgroup.critical.com/v1
kind: RTResource
metadata:
  name: rt-critical-application
  namespace: realtime
spec:
  namespace: realtime
  replicas: 4
  selector:
    matchLabels:
      app-selector: rt-critical-app
  criticality: 1
  template:
    metadata:
      name: rt-critical-pod
      namespace: realtime
      labels:
        app-selector: rt-critical-app
      annotations:
        app-annotation: rt-critical-annotation
    spec:
      containers:
        - name: rt-critical-container
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "700m"
              memory: "200Mi"
            limits:
              cpu: "700m"
              memory: "200Mi"
```

**Key Configuration**:
- **`criticality: 1`**: Highest priority level (lower number = higher priority)
- **`replicas: 4`**: Desired number of pod replicas
- **`selector.matchLabels`**: Labels used to identify pods managed by this RTResource
- **`template`**: Pod template specification (similar to Deployment template)
- **Resources**: Fixed CPU (700m) and memory (200Mi) allocation

**Usage**:
```bash
# Enter the current working directory
kubectl apply -f application-example/test-application.yaml

# Verify RTResource creation
kubectl get rtresource -n realtime

# Check managed pods
kubectl get pods -n realtime

# Change the number of replicas in RTResource manifest
# Apply the updated manifest to trigger scaling
kubectl apply -f application-example/test-application.yaml

# Delete the RTResource to clean up all pods
kubectl delete -f application-example/test-application.yaml
```

## Knative Example

### [`knative-application-example/knative-test-application.yaml`](knative-application-example/knative-test-application.yaml)

Example of a critical serverless application using Knative with Preempt-K8s integration.

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: knative-critical-application
  namespace: realtime
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/metric: concurrency
        autoscaling.knative.dev/target: "10"
        autoscaling.knative.dev/application-criticality-level: "1"
    spec:
      containers:
        - name: knative-critical-container
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "700m"
              memory: "200Mi"
            limits:
              cpu: "700m"
              memory: "200Mi"
```

**Key Configuration**:
- **`autoscaling.knative.dev/metric: concurrency`**: Scale based on concurrent requests
- **`autoscaling.knative.dev/target: "10"`**: Target 10 concurrent requests per pod
- **`autoscaling.knative.dev/application-criticality-level: "1"`**: Criticality level for the PREEMPT-FaaS (requires the patched version of Knative)
- **Resources**: Same as RTResource example for consistent comparison

**Prerequisites**:
- Knative Serving installed
- Patched Knative version with RTResource integration (see [main README](../../../README.md#knative-integration))

**Usage**:
```bash
kubectl apply -f knative-application-example/knative-test-application.yaml

# Verify Knative Service
kubectl get ksvc -n realtime

# Check generated RTResource (created by Knative)
kubectl get rtresources -n realtime

# Delete the Knative Service to clean up all resources
kubectl delete -f knative-application-example/knative-test-application.yaml
```

Check the [README.md](../test-pods/README.md) to learn how to stress the application and trigger scaling operations.

### Audit Log Patterns

The subdirectories [`apiserver-kube-manager-auditing-flux`](knative-application-example/apiserver-kube-manager-auditing-flux/) and [`apiserver-preempt-k8s-auditing-flux/`](knative-application-example/apiserver-preempt-k8s-auditing-flux) contain Kubernetes audit log JSON files representing the **Kube-Manager** and **PREEMPT-FaaS** interactions with the **Kube-apiserver** when orchestrating resources.
