# Loki Installation

Quick installation guide for Loki log aggregation system.

## Configuration

The [`loki-deploy.yaml`](loki-deploy.yaml) file contains:
- Single binary deployment mode with 1 replica
- Node selector for specific worker node (update it to deploy it on the worker node you want)
- Persistent storage enabled (10Gi volume)
- 7 days log retention period (`retention_period: 168h`)
- OTLP configuration for receiving logs from OpenTelemetry Collector
- Structured metadata support enabled
- Gateway and MinIO disabled (filesystem storage)
- Resource limits: 500m CPU, 512Mi memory

Edit this file to customize the deploy before installation.

## Installation

Install Loki with our custom configuration:

```bash
# Enter the grafana-loki folder
cd monitoring/grafana-loki

# Install Loki with Helm using our custom values
helm install loki ./loki --values loki-deploy.yaml -n observability
```

### Verify Installation

Check that Loki is running:

```bash
kubectl get pods -n observability | grep loki
kubectl get svc loki -n observability
```

Verify the persistent volume:

```bash
kubectl get pvc -n observability | grep loki
```

## Access Loki

Loki provides HTTP API for querying logs. Forward the port:

```bash
kubectl port-forward svc/loki 3100:3100 -n observability
```

Query logs via HTTP:

```bash
curl http://localhost:3100/loki/api/v1/labels
```

## Uninstall

Remove Loki:

```bash
helm uninstall loki -n observability
```

**Note**: The persistent volume and its claim will remain. To delete them, remove the claim:

```bash
kubectl delete pvc -n observability storage-loki-0
```