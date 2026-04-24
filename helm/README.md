# PREEMPT-FaaS Helm Chart

This Helm Chart allows you to quickly configure and install **PREEMPT-FaaS** in your K8s cluster.

## Prerequisites (recommended)

- Kubernetes cluster v1.29+ with admin access
- Helm 3.20+
- A control-plane node that allows Pod scheduling (only single-node control plane has been tested)

## Installed Components

The Chart installs the following components:

- **Custom Resource Definition (CRD)**: defines the structure of the custom resources managed by PREEMPT-FaaS, called **RTResources**, each representing a critical application;
- **Service Account**: dedicated service account for the controller;
- **ClusterRole and ClusterRoleBinding**: grant permissions to manage RTResources and Pods to the controller Service Account;
- **ConfigMap**: the controller configuration encoded in Container Environment Variables;
- **StatefulSet**: PREEMPT-FaaS deploy as a single-replica StatefulSet.

## Configuration

The [`values.yaml`](values.yaml) file contains all configurable parameters of the Chart. Users can customize their PREEMPT-FaaS installation by changing these parameters directly in this file, or creatind a new file containing only the parameters to override (Helm will merge them with the rest of the default configuration). Below is a detailed description of each field.

### General Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.general.name` | Controller application name | `preempt-k8s` |
| `preempt_k8s.general.namespace` | Namespace where the controller will be deployed | `realtime` |

### Pod Configuration

#### Restart Policy

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.pod.restartPolicy` | Restart policy for the controller pod | `Always` |

#### Resources

CPU and memory resources allocated to the controller:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.pod.resources.limits.cpu` | Maximum CPU limit | `2` cores |
| `preempt_k8s.pod.resources.limits.memory` | Maximum memory limit | `3Gi` |
| `preempt_k8s.pod.resources.requests.cpu` | Requested (guaranteed) CPU | `2` cores |
| `preempt_k8s.pod.resources.requests.memory` | Requested (guaranteed) memory | `2Gi` |

#### Security Context

The controller requires elevated privileges to manage scheduling policy and real-time priorities:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.pod.securityContext.runAsUser` | User UID for the container | `0` (root) |
| `preempt_k8s.pod.securityContext.runAsGroup` | Group GID for the container | `0` |
| `preempt_k8s.pod.securityContext.fsGroup` | Filesystem GID | `0` |
| `preempt_k8s.pod.securityContext.allowPrivilegeEscalation` | Allow privilege escalation | `true` |
| `preempt_k8s.pod.securityContext.privileged` | Run container in privileged mode | `true` |
| `preempt_k8s.pod.securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `false` |

##### Capabilities

Linux capabilities required for real-time operations:

| Capability | Description |
|------------|-------------|
| `SYS_NICE` | Modify process scheduling priorities and policies |
| `IPC_OWNER` | Manage IPC queues and shared memory |
| `SYS_ADMIN` | Various administrative operations |
| `MKNOD` | Create special files |
| `SYS_RESOURCE` | Override resource limits |

#### Container Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.pod.container.name` | Container name | `preempt-k8s-controller` |
| `preempt_k8s.pod.container.image.repository` | Docker image repository | `dessertunina/preempt-k8s-cpu-pinned` |
| `preempt_k8s.pod.container.image.tag` | Docker image tag | `1.0.0` |
| `preempt_k8s.pod.container.image.pullPolicy` | Image pull policy | `Always` |
| `preempt_k8s.pod.container.port` | Container exposed port | `80` |

**Note**: The default image points to our private repository, build the controller and replace the image name and tag. To build the controller, follow the [provided guide](../controller/README.md).

#### Controller Configuration (ConfigMap)

These parameters configure the runtime behavior of the controller:

##### Watchdog Management

Watchdogs are worker threads that handle events for critical applications:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.configMap.MIN_WATCHDOGS` | Minimum number of active watchdogs | `10` |
| `preempt_k8s.configMap.MAX_WATCHDOGS` | Maximum number of active watchdogs | `20` |
| `preempt_k8s.configMap.THRESHOLD` | Idle watchdogs threshold for dynamic thread scaling | `3` |
| `preempt_k8s.configMap.EVENT_QUEUE` | Shared event queue path | `/eventqueue` |

**How it works**: when the number of idle watchdogs drops under `THRESHOLD`, the scale-up is triggered, and, conversely, scale-down is performed when the `THRESHOLD` is below the number of available watchdogs. The total watchdog count is limited in the range [ `MIN_WATCHDOGS` ; `MAX_WATCHDOGS` ].

##### CPU Pinning

CPU pinning allows assigning specific controller threads to dedicated CPU cores to guarantee predictable real-time performance:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `preempt_k8s.configMap.THREAD_CPU_PINNING` | Enable/disable CPU pinning | `false` |
| `preempt_k8s.configMap.RESOURCE_WATCHER_CPU_LIST` | CPU cores for Resource Watcher thread | `0` |
| `preempt_k8s.configMap.POD_WATCHER_CPU_LIST` | CPU cores for Pod Watcher thread | `1` |
| `preempt_k8s.configMap.SERVER_CPU_LIST` | CPU cores for Event Server thread | `2` |
| `preempt_k8s.configMap.STATE_UPDATER_CPU_LIST` | CPU cores for State Updater thread | `3` |
| `preempt_k8s.configMap.WATCHDOGS_CPU_LIST` | CPU cores for Watchdog threads | `4,5` |

**Note**: 
- Values are lists of CPU cores (e.g., `[0]`, `[1,2,3]`)
- Ensure the specified cores are available on the control-plane node
- Default configuration assumes a system with at least 6 available cores
- To disable CPU pinning, set `THREAD_CPU_PINNING` to `false`

##### Thread Descriptions

- **Resource Watcher**: monitors changes to RTResources
- **Pod Watcher**: monitors the state of RTResources' Pods
- **Server**: handles watchdogs scale-up
- **State Updater**: updates the state RTResources according to Pod availability
- **Watchdogs**: worker threads that manage the lifecycle of critical applications

## Installation

To install the Chart with the release name `preempt-k8s`:

```bash
# From the project root folder
# Ensure the namespace is the same reported in the 'values.yaml' or its override file

# Create the namespace if it does not exist in the cluster
kubectl create namespace <namespace-name>

# Install the Helm Chart
helm install preempt-k8s ./helm -n <namespace-name>
```

The **StatefulSet** will create a pod called `preempt-k8s-0`.

### Installation with Custom Configuration

You can override default values by creating a `custom.yaml` file:

```bash
helm install preempt-k8s ./helm -n <namespace-name> --values <path-to-custom.yaml>
```

Or by specifying individual parameters:

```bash
helm install preempt-k8s ./helm -n <namespace-name> \
  --set preempt_k8s.configMap.MIN_WATCHDOGS=15 \
  --set preempt_k8s.configMap.MAX_WATCHDOGS=25
```

### Installation Verification

After installation, verify that the CRD was created:

```bash
kubectl get crd rtresources.rtgroup.critical.com
```

Verify that the controller is running:

```bash
kubectl get statefulset -n <namespace-name>
kubectl get pods -n <namespace-name>
kubectl logs -n <namespace-name> preempt-k8s-0 -f
```

Check the following folder for usage examples: [Application examples](../experiments/examples/).

Check the root project docs for information about Knative compliant with PREEMPT-FaaS: [Main project README](../README.md).

### Configuration Update

To update the configuration of an existing installation:

```bash
helm upgrade preempt-k8s ./helm -n <namespace-name> --values custom.yaml
```

Or by modifying individual parameters:

```bash
helm upgrade preempt-k8s ./helm -n <namespace-name> \
  --set preempt_k8s.configMap.MIN_WATCHDOGS=15
```

## Uninstall Procedure

To completely remove the controller and all associated resources:

```bash
helm uninstall preempt-k8s -n <namespace-name>
```

If you want to remove the namespace entirely:

```bash
kubectl delete namespace <namespace-name>
```
