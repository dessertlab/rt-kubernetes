# OpenTelemetry Collector Installation

Quick installation guide for OpenTelemetry Collector configured to collect Kubernetes audit logs.

## Configuration

The [`opentelemetry-collector-deploy.yaml`](opentelemetry-collector-deploy.yaml) file contains:
- Deployment mode with single replica
- Node selector for control-plane node (update with your control plane node, the [README.md](../../helm/README.md) Pod configuration section can help you configure it)
- Filelog receiver to collect audit logs from files (see [AUDITING.md](../audit/AUDITING.md))
- OTLP HTTP exporter to send logs to Loki
- A processor sends data in batches to optimize performance every 10 seconds or when the batch reaches 200 items
- Setup of the logs pipeline-

Edit this file to customize the deploy before installation.

**Audit Logs Pipeline**
```yaml
receivers:
  pipelines:
    logs:
      receivers:
        - filelog
      processors:
        - resource
        - batch
      exporters:
        - otlphttp
```

**Note**:

- The `audit log` file is mounted as a volume
- It runs as **root** user
- Default receivers are disabled: zipkin, prometheus, jaeger.
- The `requestReceivedTimestamp` of each audit log is set as the log timestamp, so that logs are correctly ordered in **Loki**
- Logs are marked as belonging to the `kubernetes-audit` job
- Unnecessary ports are disabled

## Installation

Install the OpenTelemetry Collector with our custom configuration:

```bash
# Enter the open-telemetry-collector folder
cd monitoring/open-telemetry-collector

# Install OpenTelemetry Collector with Helm using our custom values
helm install otel-collector ./opentelemetry-collector --values opentelemetry-collector-deploy.yaml -n observability
```

### Verify Installation

Check that the OpenTelemetry Collector is running:

```bash
kubectl get pods -n observability | grep "opentelemetry"
kubectl get svc -n observability | grep "opentelemetry"
```

Check the logs to verify it's collecting audit logs (only error will appear):

```bash
kubectl logs -n observability deployment/otel-collector-opentelemetry-collector -f
```

## Uprgade the Configuration

To update the OpenTelemetry Collector configuration:

- 1. Edit the [`opentelemetry-collector-deploy.yaml`](opentelemetry-collector-deploy.yaml) file with your changes

- 2. Upgrade the Helm release with the new configuration:
```bash
helm upgrade otel-collector ./opentelemetry-collector --values opentelemetry-collector-deploy.yaml -n observability
```

## Uninstall

Remove OpenTelemetry Collector:

```bash
helm uninstall otel-collector -n observability
```
