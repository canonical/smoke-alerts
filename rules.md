# All alert rules
An overview of all alert rules.

## Charmed OpenStack Alert Rules

| Alert Name | Trigger Condition | Severity |
|------------|-------------------|----------|
| **CharmedOpenStackAPIFailureRateHigh** | API failure rate >20% across all services over 1 hour | Warning |
| **CharmedOpenStackAPIFailureRateCritical** | API failure rate >70% across all services over 1 hour | Critical |
| **CharmedOpenStackKeystoneAPIFailure** | Keystone API failure rate >10% over 5 minutes | Warning |
| **CharmedOpenStackNovaAPIFailure** | Nova API failure rate >15% over 10 minutes | Warning |
| **CharmedOpenStackNeutronAPIFailure** | Neutron API failure rate >15% over 10 minutes | Warning |
| **CharmedOpenStackGlanceAPIFailure** | Glance API failure rate >15% over 10 minutes | Warning |
| **CharmedOpenStackNewRoleAssignment** | New role assignment detected | Warning |
| **CharmedOpenStackHighLoginFailureRate** | Authentication failure rate >20% over 1 hour | Warning |
| **CharmedOpenStackMultipleFailedLoginsWarning** | User has ≥3 failed login attempts in 10 minutes | Warning |
| **CharmedOpenStackMultipleFailedLoginsCritical** | User has ≥5 failed login attempts in 10 minutes | Critical |

