# Experiments

This directory contains experimental configurations, benchmarks, and utilities used for testing and evaluating the PREEMPT-FaaS controller under various workload scenarios.

## Directory Structure

```
experiments/
├── examples/                           # Example application manifests
│   ├── application-example/            # Basic RTResource example
│   └── knative-application-example/    # Knative service example employing PREEMPT-FaaS
├── exempt-configuration/               # API Priority and Fairness configurations
├── interfering-load/                   # Scripts for generating stress workload
├── k8s-results-store/                  # Persistent storage setup for experiment results
├── test-pods/                          # Test pods with benchmarking tools
└── vSwarm-benchmarks/                  # Serverless benchmarks from the vSwarm suite
```

## Prequisites

Before running the examples, the experiments, and the benchmarks provided in these folders, make sure to setup the volume provisioners first, as describedi in [LONGHORN.md](./k8s-results-store/LONGHORN.md).

Once the provisioner is up and running, with our experiments storage deployed, setup the monitoring infrastructure, included in the [`monitoring`](../monitoring/) folder, as described in [MONITORING.md](../monitoring/MONITORING.md).

## Knative Benchmark

Our patched version of Knative also includes a modified version of the `real-traffic-test` performance benchmark. Once you set up the environment, take a look at the repository at: https://github.com/dessertlab/knative-crd-scaled/tree/knative-patch/test/performance/benchmarks/real-traffic-test.
