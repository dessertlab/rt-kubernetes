# Interfering Load Generator

Script to generate continuous interfering workload for stress testing **PREEMPT-FaaS** and **Kube-Manager** controllers by repeatedly creating and deleting resources on a target node.

This script is mostly employed in our experiments and never executed on its own.

## Usage

```bash
./interfering-load.sh [-t <type>] [-i <number>] [-c <criticality>] [-n <node>]
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `-t <type>` | Resource type: `Deployment` or `RTResource` | `Deployment` |
| `-i <number>` | Number of resources per burst | `1` |
| `-c <level>` | Application criticality level (RTResource only) | `2` |
| `-n <node>` | Target bucket node name | `dessertw3` |
| `-h` | Show help message | - |

## Behavior

The script runs in an **infinite loop**:

1. Creates `N` resources simultaneously (burst)
2. Deletes all created resources (burst)
3. Repeats indefinitely

Each RTResource/Deployments resource creates a single Pod through the respective controllrs:
- **Image**: `nginx:latest`
- **CPU**: 700m (0.7 cores)
- **Memory**: 200Mi
- **Namespace**: `interference` (auto-created)
- **Node**: Specified with `-n` flag

## Stopping the Script

Press **Ctrl+C** to stop. The script will automatically clean up all created resources.

## Use Cases

- **Controller Stress Testing**: Measure Preempt-K8s performance under high API load
- **Scaling Interference**: Test critical application behavior during cluster churn
- **API Server Load**: Evaluate API server responsiveness with continuous resource changes
- **Comparative Testing**: Compare Deployment vs RTResource handling under stress

## Prerequisites

- kubectl configured with cluster access
- Target node must exist in the cluster
- Sufficient permissions to create/delete resources
- For RTResource mode: PREEMPT-FaaS controller installed

## Notes

- All resources are created in the `interference` namespace
- Pods are deployed on a specific bucket node, this choice was made to isolate the interference in the worker nodes hosting actual services
- Cleanup is automatic on script termination (Ctrl+C)
- The script validates inputs before starting
- Selecting a single node for deploy will limit the number of Pods actually started on the node, this is not a limitation since the goal is to stress the Kube-apiserver and the controllers, not the node itself
