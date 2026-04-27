# API Exempt Configuration

FlowSchema configuration to exempt critical application API requests from Kube-apiserver rate limiting and queuing using the **Priority and Fairness** feature.

More details in the Kubernetes official docs: https://kubernetes.io/docs/concepts/cluster-administration/flow-control/

## Overview

This configuration grants the `exempt` priority level to API requests related to critical applications, ensuring they are never throttled by the API server even under heavy load.

## Configuration File

```yaml
apiVersion: flowcontrol.apiserver.k8s.io/v1
kind: FlowSchema
metadata:
  name: critical-application
spec:
  matchingPrecedence: 3
  priorityLevelConfiguration:
    name: exempt
  rules:
  - resourceRules:
    - apiGroups:
      - ""
      - apps
      - rtgroup.critical.com
      clusterScope: false
      namespaces:
      - default
      resources:
      - pods
      - pods/status
      - deployments
      - deployments/status
      - rtresources
      - rtresources/status
      verbs:
      - '*'
    subjects:
    - group:
        name: system:authenticated
      kind: Group
```

**Key Configuration**:
- **`priorityLevelConfiguration: exempt`** - Highest priority level, never throttled
- **`matchingPrecedence: 3`** - Priority for matching (lower value = higher precedence)
- **API Groups**: Covers core API, apps, and RTResource custom resources
- **Namespace**: `default` (modify for your critical application namespace)
- **Resources**: Pods, Deployments, RTResources and their status
- **Subjects**: All authenticated and unauthenticated users

## Installation

Apply the FlowSchema configuration:

```bash
kubectl apply -f api-exempt.yaml
```

### Verify Installation

Check that the FlowSchema was created:

```bash
kubectl get flowschema
```

## Uninstall

Remove the FlowSchema:

```bash
kubectl delete -f api-exempt.yaml
```

### Verify Uninstallation

Check that the FlowSchema was deleted:

```bash
kubectl get flowschema
```

## Note

This configuration was only used for a one-time test. We do not need to rely on it.
