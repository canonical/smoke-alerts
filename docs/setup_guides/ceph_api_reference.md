# Ceph RESTful API (Ceph API) Failure Rate Alerts

## Overview
The Ceph Manager (MGR) exposes RESTful API (which is known as **Ceph API**) via its Ceph Dashboard module for managing and monitoring the Ceph cluster. Notice that this "Ceph API" is only for Ceph cluster management (check cluster health, view OSD status, create pools, manage users, etc.,) and does not handle data operations (RBD Operations, Object Storage Operations, File Storage Operations). Ceph RADOS Gateway (RGW) exposes its own RESTful API for data operations, while Ceph RBD and CephFS use the native Ceph protocol (RADOS) for data operations instead of RESTful APIs.

The scope of the alert rules in [ceph_dashboard_rules.rule](../rules/prod/loki/ceph_dashboard_rules.rule) is to monitor the API access failure rates (via HTTP status codes) for a [charmed Ceph cluster](https://ubuntu.com/ceph/docs) using the Ceph API, which is only for control plane operations rather than data plane operations.

> **Note:** The Ceph dashboard module is optional. If you do not plan to use the Ceph dashboard for managing and monitoring the Ceph cluster, these alert rules will not be applicable.

**Reference:**
- [Ceph RESTful API Documentation](https://docs.ceph.com/en/latest/mgr/ceph_api/)
- [Ceph Dashboard](https://docs.ceph.com/en/latest/mgr/dashboard/#ceph-dashboard)

## Pre-requisites
- [COS](https://documentation.ubuntu.com/observability/latest/) deployment
- The charmed Ceph cluster should already be managed and monitored via the Ceph Dashboard module. This means the [ceph-dashboard charm](https://charmhub.io/ceph-dashboard) is already deployed and related to [ceph-mon charm](https://charmhub.io/ceph-mon). If not, and now the admin wants to manage the Ceph cluster using the Ceph Dashboard, this can be done following this [guide](https://ubuntu.com/ceph/docs/install-dashboard#deployment). To quickly verify the dashboard is enabled:
    ```bash
    $ juju exec -u ceph-mon/leader -- ceph mgr module ls | grep dashboard
    # Expected output:
    dashboard          on

    $ juju exec -u ceph-mon/leader -- ceph mgr services | grep dashboard
    # Expected output:
    "dashboard": "https://10.149.54.80:8443/",
    ```
    Refer to the _**Ceph RESTful API Documentation**_ above for the detailed usage of Ceph API.

## Setup Steps

### 1. Deploy and Configure Grafana Agent
Deploy Grafana Agent and connect to COS:
```bash
# Deploy grafana-agent
juju deploy grafana-agent

# Relate to ceph-mon
juju relate grafana-agent ceph-mon

# Connect to COS (assumes offers exist)
juju relate grafana-agent loki-logging
```

### 2. Configure COS Alert Rules
Switch to your COS model and deploy [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s):

```bash
# Switch to COS model
juju switch cos

# Deploy cos-configuration-k8s charm
juju deploy cos-configuration-k8s cos-config \
    --config git_repo=https://github.com/canonical/smoke-alerts \
    --config git_branch=main \
    --config git_depth=1 \
    --config loki_alert_rules_path=rules/prod/loki/ \

# Relate to Loki and Prometheus
juju relate loki cos-config
```
After this point, the alert rules in [ceph_dashboard_rules.rule](../rules/prod/loki/ceph_dashboard_rules.rule) will be applicable.

## API Log Source and Format
The HTTP access logs for Ceph API are recorded in `/var/log/ceph/ceph-mgr.*.log` file in the `ceph-mon` machine. Example log entries:
```log
2026-01-09T01:06:40.013+0000 7f7b3f7ee640  0 [dashboard INFO request] [::ffff:10.149.54.71:55826] [POST] [400] [0.150s] [85.0B] [0f83d360-51db-4bad-aa44-0269b71aef10] /api/auth
2026-01-09T01:33:34.696+0000 7f7b3b7e6640  0 [dashboard INFO request] [::ffff:10.149.54.71:60974] [POST] [201] [0.380s] [1.3K] [dbcce1e9-76f6-40d5-a66f-70e4098915d3] /api/auth
2026-01-09T01:34:48.002+0000 7f7b3f7ee640  0 [dashboard INFO request] [::ffff:10.149.54.71:41564] [GET] [200] [0.012s] [admin] [1.3K] /api/health/minimal
2026-01-09T02:00:50.025+0000 7f7b3e7ec640  0 [dashboard INFO request] [::ffff:10.149.54.71:56588] [GET] [200] [0.011s] [admin] [37.9K] /api/pool
2026-01-09T02:02:40.531+0000 7f7b3d7ea640  0 [dashboard INFO request] [::ffff:10.149.54.71:59216] [GET] [200] [0.007s] [admin] [14.1K] /api/osd
2026-01-09T02:11:41.658+0000 7f7b3c7e8640  0 [dashboard INFO request] [::ffff:10.149.54.71:33288] [POST] [201] [1.640s] [admin] [4.0B] /api/pool
2026-01-09T02:11:50.548+0000 7f7b3f7ee640  0 [dashboard INFO request] [::ffff:10.149.54.71:44014] [PUT] [200] [1.558s] [admin] [4.0B] /api/pool/test-api-pool
2026-01-09T02:14:49.488+0000 7f7b3b7e6640  0 [dashboard INFO request] [::ffff:10.149.54.71:38614] [GET] [200] [0.008s] [admin] [39.9K] /api/pool
2026-01-09T02:15:42.429+0000 7f7b3f7ee640  0 [dashboard INFO request] [::ffff:10.149.54.71:44018] [DELETE] [400] [0.025s] [admin] [182.0B] /api/pool/test-api-pool
2026-01-09T02:18:38.259+0000 7f7b3c7e8640  0 [dashboard INFO request] [::ffff:10.149.54.71:53300] [GET] [404] [0.007s] [admin] [513.0B] /api/random
2026-01-09T02:18:49.095+0000 7f7b3b7e6640  0 [dashboard INFO request] [::ffff:10.149.54.71:45060] [GET] [200] [0.006s] [admin] [73.0B] /api/osd/flags
```



