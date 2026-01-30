# Charmed Kubeflow Monitoring Setup

This guide covers a sample set up for monitoring gateway API for a Charmed Kubeflow deployment integrated with Canonical Observability Stack (COS).

## Overview

Charmed Kubeflow uses Istio as its ingress gateway. The `istio-ingressgateway` handles all external user traffic to Kubeflow services (dashboards, notebooks, pipelines). The gateway logs capture user-facing HTTP requests to dashboards, notebooks, and pipelines. By monitoring this gateway's access logs, we can detect user-facing API failures.

## Prerequisites

- [Charmed Kubeflow](https://documentation.ubuntu.com/charmed-kubeflow/) deployment
- [COS](https://documentation.ubuntu.com/observability/latest/) deployment
- COS integrated with Kubeflow (see [official guide](https://documentation.ubuntu.com/charmed-kubeflow/how-to/manage/integrate-cos/))

## Setup Steps

### 1. Integrate COS with Kubeflow

Follow the [official COS integration guide](https://documentation.ubuntu.com/charmed-kubeflow/how-to/manage/integrate-cos/) to set up basic monitoring.

This establishes `grafana-agent-k8s` in the Kubeflow model with relations to COS.

### 2. Deploy Promtail for Non-Sidecar Charms

The istio-ingressgateway uses a non-sidecar pattern and requires Promtail for log forwarding. See [official documentation](https://documentation.ubuntu.com/charmed-kubeflow/reference/monitoring/loki-logs/#non-sidecar-pattern-charms).

#### Get Required Configuration Values
In your Juju model that deploys Kubeflow, run:
```bash
# Get Loki URL
juju show-unit grafana-agent-k8s/0 --endpoint logging-consumer \
  | yq '.[]."relation-info".[]."related-units".[].data.endpoint | fromjson | .url'

# Get Kubeflow model UUID
juju models --format json \
  | jq -r '.models[] | select(."short-name"=="kubeflow") | ."model-uuid"'
```

#### Deploy Promtail
> **Note:**
This example below follows the official guide to forward logs from the `istio-ingressgateway` and `istiod`. However, only `istio-ingressgateway` logs are used for alerting in this guide, so an actual deployment may only need that.

Create `CKF_promtail.yaml` with the manifests below, updating the `juju_model`, `url` and `juju_model_uuid` values:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: promtail
  labels:
    app: promtail
spec:
  selector:
    matchLabels:
      app: promtail
  template:
    metadata:
      labels:
        app: promtail
    spec:
      serviceAccount: promtail-serviceaccount
      containers:
        - name: promtail
          image: grafana/promtail
          args:
            - -config.file=/etc/promtail/promtail.yaml
          env:
            - name: 'HOSTNAME'
              valueFrom:
                fieldRef:
                  fieldPath: 'spec.nodeName'
          volumeMounts:
            - name: logs
              mountPath: /var/log/pods
            - name: promtail-config
              mountPath: /etc/promtail
      volumes:
        - name: logs
          hostPath:
            path: /var/log/pods
        - name: promtail-config
          configMap:
            name: promtail-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: promtail-config
data:
  promtail.yaml: |
    server:
      http_listen_port: 9080
      grpc_listen_port: 0
    clients:
      - url: <LOKI_URL>
        external_labels:
          juju_model: <KUBEFLOW_MODEL_NAME>
          juju_model_uuid: <KUBEFLOW_MODEL_UUID>
    positions:
      filename: /tmp/positions.yaml
    target_config:
      sync_period: 10s
    scrape_configs:
      - job_name: istio
        kubernetes_sd_configs:
          - role: pod
            namespaces:
              names:
                - kubeflow
            selectors:
              - role: pod
                label: "app in (istio-ingressgateway, istiod)"
        relabel_configs:
          - action: replace
            source_labels: [__meta_kubernetes_pod_container_name]
            target_label: workload
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - action: replace
            source_labels: [__meta_kubernetes_namespace]
            target_label: namespace
          - action: replace
            source_labels: [__meta_kubernetes_pod_name]
            target_label: pod
          - source_labels: [__meta_kubernetes_pod_node_name]
            target_label: __host__
          - replacement: /var/log/pods/*$1/*.log
            separator: /
            source_labels: [__meta_kubernetes_pod_uid, __meta_kubernetes_pod_container_name]
            target_label: __path__
        pipeline_stages:
          - labeldrop: [filename]
          - match:
              selector: '{app="istio-ingressgateway"}'
              stages:
                - static_labels:
                    juju_application: istio-ingressgateway
                    juju_unit: istio-ingressgateway/0
                    charm: istio-gateway
          - match:
              selector: '{app="istiod"}'
              stages:
                - static_labels:
                    juju_application: istio-pilot
                    juju_unit: istio-pilot/0
                    charm: istio-pilot
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: promtail-clusterrole
rules:
  - apiGroups: [""]
    resources: [nodes, services, pods]
    verbs: [get, watch, list]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: promtail-serviceaccount
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: promtail-clusterrolebinding
subjects:
  - kind: ServiceAccount
    name: promtail-serviceaccount
    namespace: kubeflow
roleRef:
  kind: ClusterRole
  name: promtail-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

Deploy to Kubeflow namespace:

```bash
kubectl apply -f CKF_promtail.yaml -n kubeflow
```

### 3. Configure COS Alert Rules

Forward alert rules in this repository using [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s):

```bash
juju switch cos

juju deploy cos-configuration-k8s cos-config \
    --config git_repo=https://github.com/canonical/smoke-alerts \
    --config git_branch=main \
    --config git_depth=1 \
    --config loki_alert_rules_path=rules/prod/loki/

juju relate loki cos-config
```

After that, the alerting rules from this repository under `rules/prod/loki/` will be active.

## Log Format

The istio-ingressgateway produces Envoy access logs, for example:

```
2026-01-30T04:09:28.36817375Z stdout F [2026-01-30T04:09:27.772Z] "GET /api/test-error-1769746167 HTTP/1.1" 404 - via_upstream - "-" 0 55 26 22 "10.149.59.124" "curl/8.5.0" "af2196c8-c6c5-47fa-8d8d-e9d05c9e8e17" "10.149.59.201" "10.1.91.51:8082" outbound|8082||kubeflow-dashboard.kubeflow.svc.cluster.local 10.1.91.146:53050 10.1.91.146:8080 10.149.59.124:33614 - -
```

Key fields:
- `"GET /api/endpoint HTTP/1.1"` - Request line
- `404` - HTTP status code (used for alerting)
- Response flags and timing information

## Additional Notes
Internal service-to-service communication within the cluster is not captured by the `istio-ingressgateway` logs. The alerting rules focus on user-facing API failures only.
