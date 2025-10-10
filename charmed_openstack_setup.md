# Charmed OpenStack Audit Logging Setup

This guide covers a sample setup of audit logging for a Charmed OpenStack deployment integrated with Canonical Observability Stack (COS).

## Overview

Charmed OpenStack supports audit logging through middleware configuration. This enables security monitoring by capturing audit events in CADF format for services like Keystone, Nova, Neutron, Glance, and Cinder.

## Prerequisites

- Charmed OpenStack deployment
- COS deployment

## Setup Steps

### 1. Deploy and Configure Grafana Agent

Deploy Grafana Agent matching your OpenStack charm base version:

```bash
# Deploy grafana-agent
juju deploy grafana-agent --base ubuntu@<match-openstack-charm-base>

# Connect to COS Loki (assumes offer exists)
juju relate grafana-agent loki-logging
```

### 2. Relate Grafana Agent to OpenStack Services

Connect Grafana Agent to your OpenStack components:

```bash
# Monitor OpenStack services (non-exhaustive list)
juju relate grafana-agent ceph-mon
juju relate grafana-agent ceph-osd
juju relate grafana-agent cinder
juju relate grafana-agent glance
juju relate grafana-agent keystone
juju relate grafana-agent neutron-api
juju relate grafana-agent nova-cloud-controller
juju relate grafana-agent nova-compute
juju relate grafana-agent ovn-central
juju relate grafana-agent placement
```

### 3. Enable Audit Logging

Enable audit middleware for OpenStack services:

```bash
juju config nova-cloud-controller audit-middleware=true
juju config cinder audit-middleware=true
juju config glance audit-middleware=true
juju config neutron-api audit-middleware=true
juju config heat audit-middleware=true
```

**Reference:** [Nova Cloud Controller charm documentation](https://charmhub.io/nova-cloud-controller)

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

Audit logs are collected from:

```
/var/log/apache2/*      # Most HTTP request information
/var/log/ceph/*
/var/log/cinder/*
/var/log/glance/*
/var/log/keystone/*     # Authentication and identity events
/var/log/nova/*
/var/log/neutron/*
/var/log/openvswitch/*
/var/log/ovn/*
```

## Audit Log Format

Audit logs follow the CADF (Cloud Auditing Data Federation) format:

```json
{
  "event_type": "audit.http.request",
  "timestamp": "2023-08-22T06:04:30.904397Z",
  "outcome": "success",
  "initiator": {
    "typeURI": "service/security/account/user",
    "id": "user-id",
    "name": "username",
    "host": {
      "address": "10.0.0.1",
      "agent": "python-keystoneclient"
    }
  },
  "target": {
    "typeURI": "service/security/account/project",
    "id": "project-id",
    "name": "projectname"
  }
}
```

## Additional Notes

- Audit logging is **not enabled by default** in Charmed OpenStack and must be configured per service

