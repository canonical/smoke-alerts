# All alert rules
An overview of all alert rules.

## Charmed OpenStack Alert Rules

### Prometheus Alerts

> **Note on Metrics Source:**
> Prometheus rules use metrics from the [OpenStack Exporter](https://github.com/openstack-exporter/openstack-exporter).
>
> **Important:** The OpenStack Exporter does not currently expose Neutron quota metrics. As a workaround, Neutron network quotas are **hardcoded** in [charmed_openstack_recording_rules.rule](../rules/prod/prometheus/charmed_openstack_recording_rules.rule) file as recording rules. These must be manually updated depending on your OpenStack deployment's quota settings.
>

> To verify the actual OpenStack network quotas:
> ```bash
> openstack quota show --network <project-id>
> ```
>
> **Jan 2025 Update:** OpenStack Exporter now supports Neutron quota metrics in the [latest alpha release](https://github.com/openstack-exporter/openstack-exporter/releases/tag/v1.8.0-alpha). Future versions of smoke-alerts could leverage these metrics directly, removing the need for hardcoded recording rules. This could be done when the upstream release becomes stable and [charmed-openstack-exporter](https://snapcraft.io/charmed-openstack-exporter) uses it in latest/stable channel. At that point, this update section may no longer be needed.

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CharmedOpenStackSubnetIPExhaustWarning** | Subnet IP usage >90% for 10 minutes | Warning |
| **CharmedOpenStackSubnetIPExhausted** | All IPs in subnet exhausted for 10 minutes | Critical |
| **CharmedOpenStackSubnetPoolExhaustWarning** | >90% of allocatable subnets (of a specific prefix length) used from subnet pool for 10 minutes | Warning |
| **CharmedOpenStackSubnetPoolExhausted** | All allocatable subnets (of a specific prefix length) exhausted in subnet pool for 10 minutes | Critical |
| **CharmedOpenStackNetworkQuotaWarning** | Network quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackNetworkQuotaExhausted** | Network quota fully exhausted for 10 minutes | Critical |
| **CharmedOpenStackPortQuotaWarning** | Port quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackPortQuotaExhausted** | Port quota fully exhausted for 10 minutes | Critical |
| **CharmedOpenStackRouterQuotaWarning** | Router quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackRouterQuotaExhausted** | Router quota fully exhausted for 10 minutes | Critical |
| **CharmedOpenStackFloatingIPQuotaWarning** | Floating IP quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackFloatingIPQuotaExhausted** | Floating IP quota fully exhausted for 10 minutes | Critical |
| **CharmedOpenStackSubnetQuotaWarning** | Subnet quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackSubnetQuotaExhausted** | Subnet quota fully exhausted for 10 minutes | Critical |
| **CharmedOpenStackSecurityGroupQuotaWarning** | Security group quota usage >90% for 10 minutes | Warning |
| **CharmedOpenStackSecurityGroupQuotaExhausted** | Security group quota fully exhausted for 10 minutes | Critical |

### Loki Alerts
| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CharmedOpenStackAPIFailureRateHigh** | API failure rate >20% across all services over 1 hour | Warning |
| **CharmedOpenStackAPIFailureRateCritical** | API failure rate >70% across all services over 1 hour | Critical |
| **CharmedOpenStackKeystoneAPIFailure** | Keystone API failure rate >20% over 10 minutes | Warning |
| **CharmedOpenStackNovaAPIFailure** | Nova API failure rate >20% over 10 minutes | Warning |
| **CharmedOpenStackNeutronAPIFailure** | Neutron API failure rate >20% over 10 minutes | Warning |
| **CharmedOpenStackGlanceAPIFailure** | Glance API failure rate >20% over 10 minutes | Warning |
| **CharmedOpenStackNewRoleAssignment** | New role assignment detected | Warning |
| **CharmedOpenStackHighLoginFailureRate** | Authentication failure rate >20% over 1 hour | Warning |
| **CharmedOpenStackMultipleFailedLoginsWarning** | User has ≥3 failed login attempts in 10 minutes | Warning |
| **CharmedOpenStackMultipleFailedLoginsCritical** | User has ≥5 failed login attempts in 10 minutes | Critical |
| **CharmedOpenStackAPIRequestBurst** | 10-minute API request rate exceeds 3x the 1-hour baseline | Warning |
| **CharmedOpenStackKeystoneRequestHigh** | Keystone receiving >1000 requests/second over 10 minutes | Warning |
| **CharmedOpenStackNovaRequestHigh** | Nova receiving >1000 requests/second over 10 minutes | Warning |

## Canonical K8s Alert Rules

### Prometheus Alerts

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CanonicalK8sFailedRequestRateIncreasing** | API failure rate (4xx/5xx) >5% over 1 hour (excludes core API) | Warning |
| **CanonicalK8sFailedRequestRateHigh** | API failure rate (4xx/5xx) >20% over 1 hour (excludes core API) | Warning |
| **CanonicalK8sFailedRequestRateCritical** | API failure rate (4xx/5xx) >70% over 1 hour (excludes core API) | Critical |
| **CanonicalK8sAPIServerHighLatency** | API 99th percentile latency >1s for 10 minutes (excludes core API) | Warning |
| **CanonicalK8sCoreAPIFailedRateIncreasing** | Core API failure rate (4xx/5xx) >5% over 1 hour | Warning |
| **CanonicalK8sCoreAPIFailedRateHigh** | Core API failure rate (4xx/5xx) >20% over 1 hour | Warning |
| **CanonicalK8sCoreAPIFailedRateCritical** | Core API failure rate (4xx/5xx) >70% over 1 hour | Critical |
| **CanonicalK8sCoreAPIServerHighLatency** | Core API 99th percentile latency >1s for 10 minutes | Warning |

### Loki Alerts

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CanonicalK8sRoleCreated** | Role created in past 3 days | Warning |
| **CanonicalK8sRoleUpdated** | Role updated in past 3 days | Warning |
| **CanonicalK8sRoleDeleted** | Role deleted in past 3 days | Warning |
| **CanonicalK8sClusterRoleCreated** | ClusterRole created in past 3 days | Warning |
| **CanonicalK8sClusterRoleUpdated** | ClusterRole updated in past 3 days | Warning |
| **CanonicalK8sClusterRoleDeleted** | ClusterRole deleted in past 3 days | Warning |
| **CanonicalK8sRoleBindingCreated** | RoleBinding created in past 3 days | Warning |
| **CanonicalK8sRoleBindingUpdated** | RoleBinding updated in past 3 days | Warning |
| **CanonicalK8sRoleBindingDeleted** | RoleBinding deleted in past 3 days | Warning |
| **CanonicalK8sClusterRoleBindingCreated** | ClusterRoleBinding created in past 3 days | Warning |
| **CanonicalK8sClusterRoleBindingUpdated** | ClusterRoleBinding updated in past 3 days | Warning |
| **CanonicalK8sClusterRoleBindingDeleted** | ClusterRoleBinding deleted in past 3 days | Warning |
| **CanonicalK8sServiceAccountCreated** | ServiceAccount created in past 3 days | Warning |
| **CanonicalK8sServiceAccountUpdated** | ServiceAccount updated in past 3 days | Warning |
| **CanonicalK8sServiceAccountDeleted** | ServiceAccount deleted in past 3 days | Warning |
| **CanonicalK8sSecretCreated** | Secret created in past 3 days | Warning |
| **CanonicalK8sSecretUpdated** | Secret updated in past 3 days | Warning |
| **CanonicalK8sSecretDeleted** | Secret deleted in past 3 days | Warning |
| **CanonicalK8sDaemonSetCreated** | DaemonSet created in past 3 days | Warning |
| **CanonicalK8sDaemonSetUpdated** | DaemonSet updated in past 3 days | Warning |
| **CanonicalK8sDaemonSetDeleted** | DaemonSet deleted in past 3 days | Warning |
| **CanonicalK8sUserAccessFailuresWarning** | User has ≥3 access failures in 10 minutes | Warning |
| **CanonicalK8sUserAccessFailuresCritical** | User has ≥5 access failures in 10 minutes | Critical |
| **CanonicalK8sAPIRequestBurst** | API request rate over 10 minutes exceeds 3x the 1-hour baseline | Warning |
| **CanonicalK8sHighRequestRatePerUser** | User exceeds 50 requests/second over 10 minutes | Warning |

## Landscape Server Alert Rules

### Loki Alerts

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **LandscapeAPIFailureRateHigh** | API failure rate >20% across all services over last 1 hour | Warning |
| **LandscapeAPIFailureRateCritical** | API failure rate >70% across all services over last 1 hour | Critical |

## Charmed Kubeflow Alert Rules

### Loki Alerts

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **KubeflowGatewayFailureRateHigh** | User-facing API failure rate (4xx/5xx) >20% over 1 hour | Warning |
| **KubeflowGatewayFailureRateCritical** | User-facing API failure rate (4xx/5xx) >70% over 1 hour | Critical |

## Charmed OpenSearch Alert Rules

### Loki Alerts

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CharmedOpenSearchSecurityAuditEventsHigh** | >3 security audit events in 10 minutes | Warning |
| **CharmedOpenSearchSecurityAuditEventsCritical** | >5 security audit events in 10 minutes | Critical |

## Charmed Ceph Alert Rules

### Prometheus Alerts
| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CephOSDDownSpecific** | An OSD down for 10 minutes | Warning |
| **CephOSDDownAllOnHost** | All OSDs in a host down for 10 minutes | Critical |
| **CephOSDCommitLatencyHigh** | OSD commit latency >150ms for 10 minutes | Warning |
| **CephOSDCommitLatencyCritical** | OSD commit latency >300ms for 10 minutes | Critical |
| **CephOSDCommitLatencyOutlier** | OSD commit latency >150ms AND >2x cluster average for 10 minutes | Warning |

### Loki Alerts
| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CephAPIFailureRateHigh** | Ceph API failure rate >20% over 1 hour (excludes /api/auth) | Warning |
| **CephAPIFailureRateCritical** | Ceph API failure rate >70% over 1 hour (excludes /api/auth) | Critical |
| **CephAuthFailureRateHigh** | Ceph authentication failure rate >20% over 1 hour | Warning |
| **CephAuthFailureRateCritical** | Ceph authentication failure rate >70% over 1 hour | Critical |
