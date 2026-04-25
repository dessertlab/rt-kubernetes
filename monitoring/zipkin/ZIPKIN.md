# Zipkin Installation

Quick installation guide for Zipkin distributed tracing system.

## Configuration

The [`zipkin-deploy.yaml`](zipkin-deploy.yaml) file contains:
- Node selector for scheduling (update it to deploy it on the worker node you want)
- JVM memory settings (2GB min, 3GB max)

Edit this file to customize the deploy before installation.

## Installation

Install Zipkin with our custom configuration:

```bash
# Enter the zipkin folder
cd monitoring/zipkin

# Install Zipkin with Helm using our custom values
helm install zipkin ./zipkin --values zipkin-deploy.yaml -n observability
```

### Verify Installation

Check that Zipkin is running:

```bash
kubectl get pods -n observability | grep "zipkin"
kubectl get svc zipkin -n observability
```

## Access Zipkin GUI

Forward the Zipkin service port to access the UI:

```bash
kubectl port-forward svc/zipkin 9411:9411 -n observability
```

Then open your browser to: http://localhost:9411.

## Uprgade the Configuration

To update the Zipkin configuration:

- 1. Edit the [`zipkin-deploy.yaml`](zipkin-deploy.yaml) file with your changes

- 2. Upgrade the Helm release with the new configuration:
```bash
helm upgrade zipkin ./zipkin --values zipkin-deploy.yaml -n observability
```

## Uninstall

Remove Zipkin:

```bash
helm uninstall zipkin -n observability
```
