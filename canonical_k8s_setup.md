# Canonical K8s Audit Logging Setup

This guide covers a sample setup of audit logging for a Canonical K8s deployment integrated with Canonical Observability Stack (COS).

## Overview

Canonical K8s requires manual configuration to enable audit logging. Once enabled, audit logs capture authentication, authorization, and RBAC events for security monitoring.

**Reference:** [How to harden your Canonical Kubernetes cluster](https://documentation.ubuntu.com/canonical-kubernetes/latest/snap/howto/security/hardening/)

## Prerequisites

- [Canonical K8s](https://documentation.ubuntu.com/canonical-kubernetes/latest/) deployment via Juju
- [COS](https://documentation.ubuntu.com/observability/latest/) deployment

## Setup Steps

### 1. Create Audit Policy File

Given the `k8s` charm that includes control-plane services for administering the cluster is deployed with the name `k8s-control-plane`, create the audit policy file on each control plane node:

```bash
# Create the audit policy directory
juju exec --application k8s-control-plane -- sudo mkdir -p /var/snap/k8s/common/etc/

# Create the audit policy file
juju exec --application k8s-control-plane -- sudo sh -c 'cat >/var/snap/k8s/common/etc/audit-policy.yaml <<EOL
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
    # Log ServiceAccount and RBAC creation, deletion, updates
    - level: RequestResponse
        verbs: ["create", "update", "delete"]
        resources:
            - group: ""
                resources: ["serviceaccounts"]
            - group: "rbac.authorization.k8s.io"
                resources: ["roles", "clusterroles", "rolebindings", "clusterrolebindings"]
    # Log all authentication attempts (users)
    - level: Metadata
        verbs: ["create", "update", "delete"]
        resources:
            - group: ""
                resources: ["secrets"]
EOL'
```

**Reference:** [Kubernetes Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)

### 2. Configure Kube-apiserver

Configure the kube-apiserver to use the audit policy:

```bash
juju exec --application k8s-control-plane -- sudo sh -c 'cat >>/var/snap/k8s/common/args/kube-apiserver <<EOL
--audit-log-path=/var/log/kubernetes/audit.log
--audit-log-maxage=30
--audit-log-maxbackup=10
--audit-log-maxsize=100
--audit-policy-file=/var/snap/k8s/common/etc/audit-policy.yaml
EOL'
```

**Note:** You will need to restart the `kube-apiserver` service for changes to take effect. The following command restarts the service on each control plane node:

```bash
juju exec --application k8s-control-plane -- sudo systemctl restart snap.k8s.kube-apiserver.service
```

### 3. Deploy and Configure Grafana Agent

Deploy Grafana Agent and connect to COS:

```bash
# Deploy grafana-agent
juju deploy grafana-agent

# Relate to Kubernetes control plane
juju relate grafana-agent k8s-control-plane

# Connect to COS (assumes offers exist)
juju relate grafana-agent loki-logging
juju relate grafana-agent prometheus-receive-remote-write
```

### 4. Configure COS Alert Rules

Switch to your COS model and deploy [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s):

```bash
# Switch to COS model
juju switch cos

# Deploy cos-configuration-k8s charm
juju deploy cos-configuration-k8s cos-config \
    --config git_repo=https://github.com/my-repo/cos-config \
    --config git_branch=main \
    --config git_depth=1 \
    --config loki_alert_rules_path=rules/prod/loki/ \
    --config prometheus_alert_rules_path=rules/prod/prometheus/

# Relate to Loki and Prometheus
juju relate loki cos-config
juju relate prometheus cos-config
```

## Audit Event Example

Audit logs capture events like ClusterRole creation:

```json
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "c4e00054-421f-4efc-9aa7-bb060059a5f3",
  "stage": "ResponseComplete",
  "requestURI": "/apis/rbac.authorization.k8s.io/v1/clusterroles",
  "verb": "create",
  "user": {
    "username": "kubernetes-admin",
    "groups": ["system:masters", "system:authenticated"]
  },
  "objectRef": {
    "resource": "clusterroles",
    "name": "system:cos",
    "apiGroup": "rbac.authorization.k8s.io",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "code": 201
  }
}
```

## Additional Notes

- **Not enabled by default**: Audit logging must be manually configured on Canonical K8s
- **Memory impact**: Enabling audit logging increases kube-apiserver memory consumption ([reference](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/))
- **Audit policy customization**: Adjust the audit policy to reduce memory consumption while capturing necessary events
- **Log rotation**: Configure `audit-log-maxage`, `audit-log-maxsize`, and `audit-log-maxbackup` appropriately

