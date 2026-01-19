# Landscape Server API Log References

## Overview
The rules defined in [landscape_server_rules.rule](../../rules/prod/loki/landscape_server_rules.rule) are to monitor the API access failure rates (via HTTP status codes) of a Landscape Server using [Landscape Server logs](https://documentation.ubuntu.com/landscape/reference/logs/logs/#server-logs).

Log files with HTTP Response codes which can be used as sources for these rules:

`api.log`
`appserver.log`
`message-server.log`
`pingserver.log`
`package-upload.log`
`package-search.log`

## Prerequisites

- [Landscape Server](https://documentation.ubuntu.com/landscape/how-to-guides/landscape-installation-and-set-up/juju-installation/#) deployment via Juju
- [COS](https://documentation.ubuntu.com/observability/latest/) deployment

## Setup Steps

### 1. Deploy and Configure Grafana Agent

Deploy Grafana Agent matching your Landscape Server base version:

```bash
# Deploy grafana-agent
juju deploy grafana-agent --base ubuntu@<match-landscape-server-base>

# Relate to landscape-server
juju relate grafana-agent landscape-server

# Connect to COS (assumes offer exists)
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

# Relate to Loki
juju relate loki cos-config
```

After this point, the alert rules in [landscape_server_rules.rule](../../rules/prod/loki/landscape_server_rules.rule) will be applicable.

## API Log Source and Format
The Landscape Server API logs are located in `/var/log/landscape-server/` directory. The relevant log files include:

From `api.log`:
```
Jan 18 23:43:07 api-1 INFO  \"127.0.0.1\" - - [18/Jan/2026:23:43:06 +0000] \"GET /metrics?action=MissingAction&access_key_id=Unknown HTTP/1.1\" 404 57 \"-\" \"GrafanaAgent/v0.40.4 (static; linux; binary)\"
```
From `appserver.log`:
```
Jan 18 23:42:53 appserver-1 INFO  GET /metrics method=GET path=/metrics status=404 duration_ms=110.058 ip=127.0.0.1 proto=HTTP/1.1 length=10610 ua="GrafanaAgent/v0.40.4 (static; linux; binary)" request_id=a9cc3b7c-39fc-4b7c-8301-e346d47364d7
```
From `message-server.log`:
```
Jan 18 23:52:03 message-server-2 INFO  \"10.17.2.199\" - - [18/Jan/2026:23:52:02 +0000] \"POST /message-system HTTP/1.1\" 200 235 \"-\" \"landscape-client/24.02-0ubuntu5.7\"
```
From `pingserver.log`:
```
Jan 18 23:52:09 pingserver-2 INFO  \"::ffff:10.17.2.199\" - - [18/Jan/2026:23:52:08 +0000] \"POST /ping HTTP/1.1\" 200 15 \"-\" \"PycURL/7.45.3 libcurl/8.5.0 GnuTLS/3.8.3 zlib/1.3 brotli/1.1.0 zstd/1.5.5 libidn2/2.3.7 libpsl/0.21.2 (+libidn2/2.3.7) libssh/0.10.6/openssl/zlib nghttp2/1.59.0 librtmp/2.3 OpenLDAP/2.6.7\"
```
From `package-upload.log`:
```
Jan 18 23:43:13 package-upload-1 INFO  \"127.0.0.1\" - - [18/Jan/2026:23:43:12 +0000] \"GET /metrics HTTP/1.1\" 404 57 \"-\" \"GrafanaAgent/v0.40.4 (static; linux; binary)\"
```
From `package-search.log`:
```
Jan 19 00:45:13 package-search INFO  package-search 2026/01/19 00:45:13 127.0.0.1:45934 - /UpdateUsns 200: succeeded (took 136.314µs)
```
