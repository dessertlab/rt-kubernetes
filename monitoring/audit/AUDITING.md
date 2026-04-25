# Kubernetes Audit Logs

## Introduction

Kubernetes **auditing** features enables the cluster to report the **activities** and **interactions** with the Kube-apiserver performed by users, applications and K8s internals.

A relevant aspect regarding the audting feature is the **time precision** of the events recorded. Kubernetes has been enhanced to allow for **microsecond precision**, but, for backward compatibility, the system still supports the **second precision**.

Kubernetes **audits**, unlike usual **events**, also allow to keep track of the **payload** of requests and responses sent to/from the Kube-apiserver, also differentiating this approach form **tracing-based monitoring** that only keeps track of the request flow form/to the Kube-apiserver and Kube-etcd.

In order to retrieve logs and useful data with, at least, **the millisecond precision**, we leveraged the auditing feature to track PREEMPT-FaaS and the Kube-controller-manager interactions with the Apiserver. This also accounts for interactions with serverless infrastructures based on K8s (e.g., **Knative**).

For further information, please refer to the K8s official documentation at the following links:

- K8s Auditing feature: <https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/>
- K8s Auditing configuration: <https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/>

## Enable Kubernetes Auditing

Follow the following steps to enable the auditing feature in your cluster, assuming the Kube-apiserver is deployed as a **static pod** (i.e., a manifest file constantly monitored by the control plane Kubelet).

### Audit Policy

The first thing to do is to create an **auditing policy** to establish which resources, and relative actions related to them, to monitor and in which namespace.

The **`audit-policy.yaml.`** file is structured as follows:
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: RequestResponse
    resources:
      - group: "rtgroup.critical.com"
        resources: ["rtresources", "rtresources/status"]
    namespaces: ["default"]

  - level: RequestResponse
    resources:
      - group: "apps"
        resources: ["deployments", "deployments/status"]
    namespaces: ["default"]

  - level: RequestResponse
    resources:
      - group: ""
        resources: ["pods", "pods/status"]
    namespaces: ["default"]
    verbs: ["create", "update", "patch", "delete"]
  
  - level: None

```

We omit the `RequestReceived` stage to discard duplicate logs. We only account for requests that have been served, i.e., in `RequestResponse` stage.

The "`- level: None`" at the end serves as the default policy rule for other logs outside of this policy's scope.

Copy this file in your control plane. The usual path would be the following: `/etc/kubernetes/audit-policy.yaml`.

```bash
# Copy the auditing policy file to the correct path
mv ./audit-policy.yaml /etc/kubernetes/audit-policy.yaml
```

### Activating the Auditing Feature

Now create a directory to store your logs.

```bash
sudo mkdir -p <path-to-your-k8s-log-dir>
sudo chmod 755 <path-to-your-k8s-log-dir>
```

The usual log path would be the following: `/var/log/kubernetes/audit`.

Now modify your kube-apiserver manifest, usually in: `/etc/kubernetes/manifests/kube-apiserver.yaml`.

Add the following command flags (replace the placeholder values with your actual configuration):

```yaml
- command:
    <...existing flags>
    - --audit-policy-file=<path-to-yout-audit-policy-file>
    - --audit-log-path=<path-to-your-k8s-log-dir>/audit.log
    - --audit-log-format=json
    - --audit-log-maxage=<your-log-retained-max-age>
    - --audit-log-maxbackup=<your-log-backup-number>
    - --audit-log-maxsize=<your-log-file-max-size-in-MB>
```

We report our configuration for reference.

```yaml
- command:
    <...existing flags>
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-format=json
    - --audit-log-maxage=7
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
```

Add the following volume mounts (replace the placeholder values with your actual configuration):

```yaml
- volumeMounts:
    <...existing volume mounts>
    - name: audit-policy
      mountPath: <path-to-yout-audit-policy-file>
      readOnly: true
    - name: audit-logs
      mountPath: <path-to-your-k8s-log-dir>
      readOnly: false
```

Add the following volumes (replace the placeholder values with your actual configuration):

```yaml
- volumes:
    <...existing volumes>
    - name: audit-policy
      hostPath:
        path: <path-to-yout-audit-policy-file>
        type: File
    - name: audit-logs
      hostPath:
        path: <path-to-your-k8s-log-dir>
        type: DirectoryOrCreate
```

Save the new configuration and the Kube-apiserver should restart itself.

Check if logs are being written in `<path-to-your-k8s-log-dir/audit.log` file after triggering an event that should be monitored (e.g., create a Pod in the `default` namespace if you are using our reference [audit-policy](./audit-policy.yaml)).
