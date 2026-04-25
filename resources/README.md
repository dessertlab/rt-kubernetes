# PREEMPT-FaaS Manual Installation

This directory contains plain Kubernetes manifests for **manual installation** of the PREEMPT-FaaS controller without using Helm. These files are the non-templated versions of the resources deployed by the Helm chart.

For automated installation and easier configuration management, we strongly recommend using the [Helm Chart](../helm/) instead.

For configuration parameters description, please refer to the [Helm Chart docs](../helm/README.md).

## Directory Structure

```
resources/
├── auth/                           # RBAC resources
│   ├── service-account.yaml        # Service Account for the controller
│   ├── role.yaml                   # Cluster Role with required permissions
│   └── role-binding.yaml           # Cluster Role Binding with controller Service Account
├── controller-deploy/              # Controller deploy resources
│   ├── configmap.yaml              # Configuration parameters
│   └── statefulset.yaml            # StatefulSet deploy
└── managed/                        # Custom Resource Definition
    └── RTResource.yaml             # RTResource CRD
```

## Prerequisites (recommended)

- A **Kubernetes cluster** v1.29+ with admin access
- **`kubectl`** configured to access your cluster
- A **control-plane node** that allows Pod scheduling (only single-node control plane has been tested)

Please refer to the main project docs for prerequisites installation docs links: [README.md](../README.md).

## Installation

Follow the following steps to manually install the controller resources in the correct order. We will use the name `preempt-k8s` for our manual release.

Before installation, you need to customize the configuration according to your environment and preferences.

### Step 1: Create your Namespace

Create the namespace where the controller will run:

```bash
# From the project root folder
# Ensure the namespace is the same reported in the manifest files
kubectl create namespace <namespace-name>
```

### Step 2: Install the Custom Resource Definition

Install the `RTResource CRD` first:

```bash
kubectl apply -f resources/managed/RTResource.yaml
```

Verify the CRD was created:

```bash
kubectl get crd rtresources.rtgroup.critical.com
```

### Step 3: Create RBAC Resources

Create the ServiceAccount, ClusterRole, and ClusterRoleBinding:

```bash
kubectl apply -f resources/auth/service-account.yaml
kubectl apply -f resources/auth/role.yaml
kubectl apply -f resources/auth/role-binding.yaml
```

Verify the RBAC resources:

```bash
kubectl get serviceaccount -n <namespace-name> preempt-k8s
kubectl get clusterrole preempt-k8s
kubectl get clusterrolebinding preempt-k8s
```

### Step 4: Create ConfigMap

Deploy the configuration:

```bash
kubectl apply -f resources/controller-deploy/configmap.yaml
```

Verify the ConfigMap:

```bash
kubectl get configmap -n <namespace-name> preempt-k8s
```

### Step 5: Deploy Controller

Finally, deploy the controller StatefulSet:

```bash
kubectl apply -f resources/controller-deploy/statefulset.yaml
```

### Step 6: Verify Installation

Check that the controller pod is running:

```bash
kubectl get statefulset -n <namespace-name>
kubectl get pods -n <namespace-name>
```

Check the controller logs:

```bash
kubectl logs -n <namespace-name> preempt-k8s-0 -f
```

The controller should start successfully and begin watching for RTResource objects.

## Test the Release

Check the following folder for usage examples: [Application examples](../experiments/examples/).

Check the root project docs for information about Knative compliant with PREEMPT-FaaS: [Main project README](../README.md).

## Updating Configuration

To update the controller configuration:

1. Edit the [`controller-deploy/configmap.yaml`](controller-deploy/configmap.yaml) file with your changes

2. Apply the updated ConfigMap:
   ```bash
   kubectl apply -f controller-deploy/configmap.yaml
   ```

3. Restart the controller to pick up the new configuration:
   ```bash
   kubectl rollout restart statefulset/preempt-k8s -n <namespace-name>
   ```

4. Monitor the rollout:
   ```bash
   kubectl rollout status statefulset/preempt-k8s -n <namespace-name>
   ```

## Uninstall Procedure

To completely remove the controller and all related resources:

### Step 1: Delete Controller and Configuration

```bash
kubectl delete -f resources/controller-deploy/statefulset.yaml
kubectl delete -f resources/controller-deploy/configmap.yaml
```

### Step 2: Delete RBAC Resources

```bash
kubectl delete -f resources/auth/role-binding.yaml
kubectl delete -f resources/auth/role.yaml
kubectl delete -f resources/auth/service-account.yaml
```

### Step 3: Delete Custom Resource Definition

```bash
kubectl delete -f resources/managed/RTResource.yaml
```

### Step 4: Delete Namespace (Optional)

If you want to remove the namespace entirely:

```bash
kubectl delete namespace <namespace-name>
```
