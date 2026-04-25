# Monitoring Infrastructure Deployment

This document describes the monitoring infrastructure deployment and configuration. The monitoring infrastructure is composed of several services that work together to collect, store and visualize logs and traces from the cluster.

## Prerequisites (recommended)

To deploy the monitoring infrastructure, you need:

- A **Kubernetes cluster** v1.29+ with admin access
- **`kubectl`** configured to access your cluster
- **Helm** 3.20+
- A **control-plane node** that allows Pod scheduling (only single-node control plane has been tested)
- A dedicated **worker node** hosting the monitoring services (optional but recommended for performance reasons)

Please refer to the main project docs for prerequisites installation links: [README.md](../README.md).

## Helm Installation

Each of the **monitoring services** needed is installed through the **Helm Charts**.

```bash
kubectl create namespace observability
helm install <service-name> <path-to-helm-chart> -n observability --values <path-to-additional-values>
```

Each component can be installed with their respective default parameters provided in the **values.yaml** file of each Chart. For each of the monitoring services, we provide an additional **values** file with customized parameters for our use case. You can use these files as they are or customize them according to your needs leveraging the `--values` flag during Chart deploy and upgrade procedures.

Please refer to the official services' documantations for further information:

- **Open-Telemetry**: <https://opentelemetry.io/docs/>

- **Loki**: <https://grafana.com/oss/loki/>

- **Zipkin**: <https://zipkin.io/>

- **Grafana**: <https://grafana.com/docs/>

Additional information on each Chart installation for our use-case is provided in the respective docs files in the **monitoring** directory.

## List of employed Monitoring Services

The monitoring infrastructure requires the following services:

- **OpenTelemetry Collector**: the *otel-collector* is deployed in the cluster as a *K8s Deployment* to collect metrics, logs and traces from sources and forward them to the backend monitoring services;

- **Loki**: receives logs from the *otel-collector* and can be queried from scripts and **Grafana**;

- **Zipkin**: receives traces from the *otel-collector* and can be queried from scripts; it also offers a GUI to visualize collected traces.

- **Grafana**: a GUI from where to visualized aggragated data, metrics and logs, **Loki** logs in our case.

**Note**: Check the **Grafana** official docs to set **Loki** as datasource; sometimes, it could be useful to rely on **Loki IP address** instead of **DNS resolution**.

**Note**: Altough the cluster is configured to produce tracing data, we only employ K8s logs, thus not requiring Zipkin to be installed. It was deployed in the early stages of the project development for early studies on the control plane behaviour during orchestration events.

## Kubernetes Customization and Monitoring Data Setup

To properly enable **log** and **trace** production in your Kubernetes cluster, follow the tutorial in [AUDITING.md](./audit/AUDITING.md) and [TRACING.md](./tracing/TRACING.md).
