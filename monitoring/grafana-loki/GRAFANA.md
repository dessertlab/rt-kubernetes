# Grafana Installation

Quick installation guide for Grafana visualization and dashboarding platform.

## Configuration

The [`grafana-deploy.yaml`](grafana-deploy.yaml) file contains:
- Node selector for specific worker node (update it to deploy it on the worker node you want)

Edit this file to customize the deploy before installation.

## Installation

Install Grafana with our custom configuration:

```bash
# Enter the grafana-loki folder
cd monitoring/grafana-loki

# Install Grafana with Helm using our custom values
helm install grafana ./grafana --values grafana-deploy.yaml -n observability
```

### Verify Installation

Check that Grafana is running:

```bash
kubectl get pods -n observability | grep "grafana"
kubectl get svc grafana -n observability
```

## Access Grafana GUI

Forward the Grafana service port to access the UI:

```bash
kubectl port-forward svc/grafana 3000:80 -n observability
```

Then open your browser to: http://localhost:3000

### Get Admin Password

Retrieve the default admin password:

```bash
kubectl get secret grafana -n observability -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Default username: `admin`

## Uninstall

Remove Grafana:

```bash
helm uninstall grafana -n observability
```