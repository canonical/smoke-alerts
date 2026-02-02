# Charmed OpenSearch Audit Logging Setup

This guide covers sample setup of audit logging for a Charmed OpenSearch deployment integrated with Canonical Observability Stack (COS).

## Overview

Charmed OpenSearch supports audit logging through the OpenSearch Security plugin. This enables security monitoring by capturing authentication attempts, authorization failures, and other security events.

## Prerequisites

- [Charmed OpenSearch](https://canonical-charmed-opensearch.readthedocs-hosted.com/2/) deployment
- [COS](https://documentation.ubuntu.com/observability/latest/) deployment

## Setup Steps

### 1. Deploy and Configure OpenTelemetry Collector

Deploy OpenTelemetry Collector as a subordinate charm to OpenSearch units:

> **Note:** This guide uses `opentelemetry-collector` because at the time of this writing, `grafana-agent` is EOL. The [rules](../../rules/prod/loki/charmed_opensearch_rules.rule) are catered for OpenTelemetry Collector and will not work with `grafana-agent`.

```bash
# Deploy opentelemetry-collector as subordinate
juju deploy opentelemetry-collector --base ubuntu@<match-opensearch-charm-base>

# Relate OpenSearch to opentelemetry-collector
juju relate opensearch opentelemetry-collector

# Connect to COS Loki (assumes offer exists)
juju relate opentelemetry-collector loki-logging
```

### 2. Enable Audit Logging

Configure OpenSearch to write audit logs to files (`log4j` backend):

```bash
juju exec --app opensearch -- sudo sh -c '
  KEY="plugins.security.audit.type"
  VALUE="log4j"
  FILE="/var/snap/opensearch/current/etc/opensearch/opensearch.yml"

  if grep -qE "^${KEY}: ${VALUE}$" "$FILE"; then
    echo "Already configured"
  elif grep -qE "^${KEY}:" "$FILE"; then
    sed -i -E "s|^${KEY}:.*|${KEY}: ${VALUE}|" "$FILE"
    echo "Updated existing line"
  else
    echo "${KEY}: ${VALUE}" >> "$FILE"
    echo "Added new line"
  fi
  grep "^${KEY}:" "$FILE"
'
```

### 3. Restart OpenSearch

Apply the configuration changes by restarting OpenSearch:

```bash
juju exec --app opensearch -- sudo systemctl restart snap.opensearch.daemon
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
    --config loki_alert_rules_path=rules/prod/loki/

# Relate to Loki
juju relate loki cos-config
```

## Log Sources

Audit logs are collected from OpenTelemetry Collector's shared logs mount:

```
/snap/opentelemetry-collector/.*/shared-logs/opensearch/*.json
```

These map to OpenSearch's actual log location:
```
/var/snap/opensearch/common/var/log/opensearch/*_server.json
```

## Audit Log Format

Audit logs are embedded as JSON within the `message` field of server logs:

```json
{
  "type": "server",
  "timestamp": "2026-01-29T00:42:14,120Z",
  "level": "INFO",
  "component": "audit",
  "cluster.name": "opensearch-93fo",
  "node.name": "opensearch-1.8be",
  "message": "{\"audit_cluster_name\":\"opensearch-93fo\",\"audit_node_name\":\"opensearch-1.8be\",\"audit_rest_request_method\":\"GET\",\"audit_category\":\"FAILED_LOGIN\",\"audit_request_origin\":\"REST\",\"audit_node_id\":\"ePAT6T0NRdOrqsZN--9w3Q\",\"audit_request_layer\":\"REST\",\"audit_rest_request_path\":\"/\",\"@timestamp\":\"2026-01-29T00:42:14.119+00:00\",\"audit_request_effective_user_is_admin\":false,\"audit_format_version\":4,\"audit_request_remote_address\":\"10.121.59.1\",\"audit_node_host_address\":\"10.121.59.133\",\"audit_rest_request_headers\":{\"User-Agent\":[\"curl/8.5.0\"],\"Host\":[\"10.121.59.133:9200\"],\"Accept\":[\"*/*\"]},\"audit_request_effective_user\":\"opensearch-client_5\",\"audit_node_host_name\":\"10.121.59.133\"}", "cluster.uuid": "sw8DD38gSoKSZhRpULtg3w", "node.id": "ePAT6T0NRdOrqsZN--9w3Q"
}
```

## Additional Notes

- Events with `audit_category` of `AUTHENTICATED` and `GRANTED_PRIVILEGES` are excluded from alerting to reduce noise from normal operations.

## References

- [OpenSearch Audit Logs Documentation](https://docs.opensearch.org/latest/security/audit-logs/index/)
- [OpenSearch Audit Log Storage Types](https://docs.opensearch.org/latest/security/audit-logs/storage-types/)
- [Charmed OpenSearch Documentation](https://canonical-charmed-opensearch.readthedocs-hosted.com/2/)
- [OpenTelemetry Collector Charm](https://charmhub.io/opentelemetry-collector)
