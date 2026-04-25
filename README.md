# PREEMPT-FaaS

**PREEMPT-FaaS** is a custom controller designed to manage mixed-criticality applications, orchestrating them according to their criticality level. It is a thread-based controller that employs preemptive `SCHED_FIFO` scheduling policy.

PREEMPT-FaaS reduces orchestration times of high-priority applications under intense orchestration workloads, with dynamic thread management ensuring a sufficient number of event handlers to manage the incoming load.

The project leverages the **Kubernetes** (a.k.a. *K8s*) native features to recreate, in a **real-time** fashion, the K8s **Deployment** lifecycle. **Custom Resources** replace Deployments, allowing us to introduce a new parameter in the application specification, its **priority level**. K8s internals are left untouched for seamless integration with the vanilla platform, supporting any version suited with the Custom Resource Definition feature.

## Project Structure

The project repository is structured as follows:
- the **controller** folder contains the Rust project implementation and the Dockerfile, along with the instructions to build the controller;
- the **helm** folder contains the *Helm Chart* employed to easly configure and deploy the controller and its related resources in the K8s cluster;
- the **resources** folder holds the same resources installed via helm, with no templating, or parametrization, for manual resource installation;
- the **monitoring** folder hosts the monitoring infrastructure setup employed in our experiments;
- the **experiments** folder holds all the experiments, scripts, tools, and utilities employed in the experimental evaluation of the project, along with the obtained raw and processed data.

We strongly invite users to consult the docs contained in each of the project folders. These files provide insight, tutorials, and useful information on the various parts our project.

**Note**: **PREEMPT-FaaS** may be also referred to as **PREEMPT-K8s**, the project previous name.

## Knative Integration

This repository includes examples and experiments designed to co-operate with a patched version of **Knative**, a serverless computing platform based on kubernetes that implements the **function-as-a-service** paradigm. Our forked knative repository including the patched version of the platfrom, developed to co-operate with PREEMPT-FaaS, can be found at:
https://github.com/dessertlab/knative-crd-scaled. Follow the instructions in the repository docs to install our Knative version.

## Prerequisites
The project requires a set of tools and platforms to operate. We link below the official docs of said tools that will guide the user through the installation process.

**Note**: Refer to the docs found in this repository folders for more detail on the tools version number required.

- Set up a Kubernetes cluster: https://kubernetes.io/docs/home/
- Knative custom version: https://github.com/dessertlab/knative-crd-scaled
- Helm: https://helm.sh/it/docs/
- kubectl compatible with the Kubernetes version installed: https://kubernetes.io/docs/home/
- Docker: https://docs.docker.com/

 
