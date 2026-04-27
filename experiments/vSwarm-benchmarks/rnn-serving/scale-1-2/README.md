# RNN Serving Scale 1→2 Benchmark

Benchmark to evaluate scale-up performance from 1 to 2 instance under interfering load.

We first give you a quick guide on how to run the experiments. Then, we will list all the parameters used in our evaluation to hel reproducibility.

## Running the benchmark

### setup-invoker-pods.sh script

| Flag | Description | Default |
|------|-------------|---------|
| `-p` | Base name of the pods to create or delete | `benchmark-pod` |
| `-b` | Base name of RNN services to create | `rnn-serving-python` |
| `-g` | Create (`true`) or delete (`false`) invoker pods | `true` |
| `-c` | Use RTResources with criticality=1 (`true`) or Deployments (`false`) | `true` |
| `-n` | Number of invoker pods and services | `5` |
| `-d` | Destination path for invoker binary and endpoints.json in pods | `/home` |
| `-s` | Invoker binary source path on local machine | `./invoker` |
| `-t` | Node where to schedule the invoker pods | `dessertw4` |
| `-l` | Comma-separated list of nodes where service pods should NOT be scheduled | `dessertw1,dessertw3,dessertw4` |
| `-h` | Show help message | - |

### experiment.sh script

| Flag | Description | Default |
|------|-------------|---------|
| `-f` | Name of the experiment | `rnn-serving-benchmark` |
| `-t` | Resource type: `Deployment` or `RTResource` | `Deployment` |
| `-i` | Invoker pod base name in default namespace | `benchmark-pod` |
| `-e` | Path to invoker binary inside the pods | `/home` |
| `-c` | Number of concurrent RNN Knative services to deploy | `5` |
| `-m` | Services manifest file base name | `kn-rnn-serving-python` |
| `-p` | Services port | `80` |
| `-q` | Requests per second per service | `50` |
| `-o` | Timeout for each request in seconds | `40` |
| `-d` | Duration of the test in seconds | `30` |
| `-n` | Number of test iterations | `1` |
| `-g` | Iterations offset | `0` |
| `-s` | Interference script to run | `interfering-load.sh` |
| `-r` | Number of interfering resources per burst | `40` |
| `-b` | Base scale for the tested services | `1` |
| `-u` | Number of allowed scale-up pods per service | `1` |
| `-l` | Namespace where the Loki service is deployed | `observability` |
| `-v` | Name of the Loki service | `loki` |
| `-a` | IP address of the Loki service for querying logs | `10.101.73.135` |
| `-k` | Flush Loki logs before each iteration (`true`/`false`) | `false` |
| `-w` | Force cleanup of services after test (`true`/`false`) | `true` |
| `-h` | Show help message | - |



### 1. Pull the images on Worker Node 2

```bash
# SSH into Worker Node 2
ssh <worker-2-user>@<worker-2-ip>

# Pull the benchmark service image
crictl pull docker.io/vhiveease/relay:latest
crictl pull docker.io/vhiveease/rnn-serving:latest
```

### 2. Setup Invoker Pods and Service Manifests

We create invoker pods and generate Knative service manifests.

All the following operations should be performed in the control plane.

```bash
# SSH into Control Plane
ssh <control-plane-user>@<control-plane-ip>

# Clone the repository if not already done
git clone https://github.com/dessertlab/preempt-k8s.git

# Navigate to the benchmark folder
cd preempt-k8s/experiments/vSwarm-benchmarks/rnn-serving/scale-1-2

# Show setup-invoker-pods.sh help message
./setup-invoker-pods.sh -h
```

**Case 1** - Setup invoker pods and generate service manifests for Deployment workflow:

```bash
# Setup invoker pods and generate service manifests for Deployment workflow
./setup-invoker-pods.sh -n <number-of-services> -c false -g true -s ../../invoker
```

**Case 2** - Setup invoker pods and generate service manifests for RTResource workflow:

```bash
# Setup invoker pods and generate service manifests for RTResource workflow
./setup-invoker-pods.sh -n <number-of-services> -c true -g true -s ../../invoker
```

### 3. Run Experiment

Execute the benchmark.

```bash
# Show experiment.sh help message
./experiment.sh -h
```

**Case 1** - Run the benchmark employing Deployments:

```bash
# Run the benchmark with Deployments
./experiment.sh -f <name-of-the-exp> -t Deployment -c <number-of-services> -n <number-of-iterations> -s ../../../interfering-load/interfering-load.sh -r <number-of-interferences>
```

Experiments will be stored in `/experiments/knative/vSwarm-benchmarks/rnn-serving/kube-manager/<name-of-the-exp>`.

**Case 2** - Run the benchmark employing RTResources:

```bash
# Run the benchmark with RTResources
./experiment.sh -f <name-of-the-exp> -t RTResource -c <number-of-services> -n <number-of-iterations> -s ../../../interfering-load/interfering-load.sh -r <number-of-interferences>
```

Experiments will be stored in `/experiments/knative/vSwarm-benchmarks/rnn-serving/preempt-k8s/<name-of-the-exp>`.

### 4. Cleanup

Remove all invoker pods and services:

```bash
# Cleanup invoker pods and Knative service manifests
./setup-invoker-pods.sh -n <number-of-services> -g false -s ../../invoker
```

## Experimental Evaluation

Our experimental evaluation was performed employing the following parameters.

### Deployment Benchmarks

| Parameter | Value |
|-----------|-------|
| Test Type | Deployment |
| Number of Services | 5 |
| RPS per service | 50 |
| Timeout | 40 seconds |
| Duration | 30 seconds |
| Iterations | 10 |
| Number of Interfering Resources | 15-30 |
| Base Scale per service | 1 |
| Scale-Ups Allowed per service | 1 |

### RTResource Benchmarks

| Parameter | Value |
|-----------|-------|
| Test Type | RTResource |
| Number of Services | 5 |
| RPS per service | 50 |
| Timeout | 40 seconds |
| Duration | 30 seconds |
| Iterations | 10 |
| Number of Interfering Resources | 15-30 |
| Base Scale per service | 1 |
| Scale-Ups Allowed per service | 1 |

## Analysis Scripts

Analysis scripts process experiment results to generate box-plots, CDFs and CSV summaries.

Create a pod to access the results storage: see [test-pods](../test-pods/README.md).

```bash
# Create a results directory for storing processed results
mkdir -p results
mkdir -p results/aggregated
mkdir -p results/compared
mkdir -p results/distributions
mkdir -p results/sensitivity-analysis

# Create a results directory to store the collected experiment data
mkdir -p results/knube-manager
mkdir -p results/preempt-k8s

# Copy the experiments results from the storage to the local machine for processing
# Example for a specific experiment
kubectl cp  test-pod:/experiments/knative/vSwarm-benchmarks/rnn-serving/kube-manager/<experiment-name> results/kube-manager/<experiment-name> -n default
cp -r /experiments/knative/vSwarm-benchmarks/rnn-serving/preempt-k8s/<experiment-name> results/preempt-k8s/ -n default
```

### results.py

Processes results from a single experiment (one controller type and interference load).

**Usage:**

```bash
python results.py <path_to_results_directory> <number_of_services> <controller_name>
```

**Example:**

```bash
# Process Deployment results
python results.py results/kube-manager/scale-1-2_30-iter_kube-manager_5-15 5 kube-manager

# Process RTResource results
python results.py results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 5 preempt-k8s
```

**Output:** Creates `processed_results/` directory inside the experiment folder with boxplots, CDF plots, and CSV files containing per-service and per-iteration metrics.

### compare-results.py

Compares per-service results between Deployment and RTResource experiments for a specific interference load.

**Usage:**

```bash
python compare-results.py <path_to_kube_manager_results> <path_to_preempt_k8s_results> <number_of_services>
```

**Example:**

```bash
python compare-results.py \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-15 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 \
  5
```

**Output:** Creates `compared/` directory in `results/` with per-service comparative boxplots and CDF plots.

### aggregated-results.py

Compares aggregated results (across all services) between Deployment and RTResource experiments for a specific interference load.

**Usage:**

```bash
python aggregated-results.py <path_to_kube_manager_results> <path_to_preempt_k8s_results> <number_of_services>
```

**Example:**

```bash
python aggregated-results.py \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-15 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 \
  5
```

**Output:** Creates `aggregated/` directory in `results/` with comparative boxplots and CDF plots aggregating all services.

### scatter-plot.py

Generates scatter plots visualizing the timeline of control plane events for an experiment.

**Usage:**

```bash
python scatter-plot.py <path_to_experiment_directory> <path_to_results_directory> <controller_name>
```

**Example:**

```bash
python scatter-plot.py \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 \
  results/distributions \
  preempt-k8s
```

**Output:** Creates scatter plots in the specified results directory showing scale-up events, pod creation, and service readiness timelines.

### sensitivity-analysis.py

Processes a sensitivity analysis on varying interfering load (15, 30 interfering resources). 45 resources per interfering burst led to unstable behaviour.

**Usage:**

```bash
python sensitivity-analysis.py \
  <path_to_kube_manager_15_results> <path_to_kube_manager_30_results> \
  <path_to_preempt_k8s_15_results> <path_to_preempt_k8s_30_results> \
  <number_of_services>
```

**Example:**

```bash
python sensitivity-analysis.py \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-15 \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-30 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-30 \
  5
```

**Output:** Creates `sensitivity-analysis/` directory in `results/` with boxplots and CDF plots showing metric variations across different interference loads, comparing the performance of the two controllers.

### all-mean-latency-cdf.py

Generate CDF plots of mean latency for sensitivity analysis.

The latency CDF is obtained with data across all services and iterations.

Other latency graphs, computed by the other scripts, obtain the CDF by averaging the latency of all requests across all services in a single iteration and, then, creating the CDF.

**Usage:**

```bash
python all-mean-latency-cdf.py \
  <path_to_kube_manager_15_results> <path_to_kube_manager_30_results> \
  <path_to_preempt_k8s_15_results> <path_to_preempt_k8s_30_results> \
  <number_of_services>
```

**Example:**

```bash
python all-mean-latency-cdf.py \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-15 \
  results/kube-manager/scale-1-2_30-iter_kube-manager_5-30 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-15 \
  results/preempt-k8s/scale-1-2_30-iter_preempt-k8s_5-30 \
  5
```

**Output:** Creates `sensitivity-analysis/` directory in `results/` with a CDF plot of mean latency across different interference loads, comparing the performance of the two controllers. In the termina, relevant statistics such as percentiles are printed to the console for each configuration.


