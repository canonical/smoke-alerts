# Log Example References

This document provides some example log lines that would trigger alert firings. These log lines are those with content indicating API failures, e.g., HTTP status codes 4xx or 5xx, or failed login attempts with "outcome": "failure", or new role assignments.

## Charmed Openstack

### CharmedOpenStackAPIFailureRate*
**Example log line:**
In `/var/log/apache2/other_vhosts_access.log`:
```
"GET /v2.1/os-security-groups HTTP/1.1" 404 523 "-" "gophercloud/v1.14.0"
```
Alert will trigger if the proportion of such lines with 4xx or 5xx status codes exceeds a threshold in a period of time.

### CharmedOpenStackKeystoneAPIFailure
**Example log line:**
In `/var/log/apache2/keystone_access.log`:
```
127.0.0.1 - - [29/Sep/2025:09:46:23 +0000] "GET /v3/auth/tokens HTTP/1.1" 404 558 "-" "python-keystoneclient"
```
Alert will trigger if the proportion of such lines with 4xx or 5xx status codes exceeds a threshold in a period of time.

### CharmedOpenStackNovaAPIFailure
**Example log line:**
In `/var/log/apache2/nova-api-os-compute_access.log`:
```
127.0.0.1 - - [29/Sep/2025:09:26:19 +0000] "GET /v2.1/os-hypervisors/detail HTTP/1.1" 404 558 "-" "gophercloud/v1.14.0"
```
Alert will trigger if the proportion of such lines with 4xx or 5xx status codes exceeds a threshold in a period of time.

### CharmedOpenStackNeutronAPIFailure
**Example log line:**
In `/var/log/neutron/neutron-server.log`:
```
2025-09-29 09:28:54.567 3312279 INFO neutron.wsgi [req-565c7bfe-ef2c-4dca-8adb-01973a48d558 18b9a79bb9d14fa595c67ee981f35158 1fcb9d1fbad0402197d275c63dc7df25 - c601551c20a445cba39b2469619c38af c601551c20a445cba39b2469619c38af] 10.149.54.41,127.0.0.1 "GET /v2.0/subnets HTTP/1.1" status: 404  len: 558 time: 0.0265512
```
Alert will trigger if the proportion of such lines with 4xx or 5xx status codes exceeds a threshold in a period of time.

### CharmedOpenStackGlanceAPIFailure
**Example log line:**
In `/var/log/apache2/glance_access.log`:
```
2025-09-30 02:36:23.513 1756847 INFO eventlet.wsgi.server [req-835482ff-112c-4d85-b9d1-b882ffd471cb 18b9a79bb9d14fa595c67ee981f35158 1fcb9d1fbad0402197d275c63dc7df25 - c601551c20a445cba39b2469619c38af c601551c20a445cba39b2469619c38af] 10.149.54.66,127.0.0.1 - - [30/Sep/2025 02:36:23] "GET /v2/images HTTP/1.1" 404 419 0.020062
```
Alert will trigger if the proportion of such lines with 4xx or 5xx status codes exceeds a threshold in a period of time.

### CharmedOpenstackNewRoleAssignment
**Example log line:**
In `/var/log/keystone/keystone.log`:
```
2025-09-30 14:01:53.549
{"message_id": "c776a61f-be8d-430d-8a57-2e7807ad1d13", "publisher_id": "identity.juju-644baf-yoga-ceph-6", "event_type": "identity.role_assignment.created", "priority": "INFO", "payload": {"typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event", "eventType": "activity", "id": "0d5683a5-2544-5eae-b25d-471003e72ae9", "eventTime": "2025-09-30T04:01:53.295798+0000", "action": "created.role_assignment", "outcome": "success", "observer": {"id": "4eade777dfc54e3a81a299b808ea4af6", "typeURI": "service/security"}, "initiator": {"id": "18b9a79bb9d14fa595c67ee981f35158", "typeURI": "service/security/account/user", "host": {"address": "10.149.54.84", "agent": "python-keystoneclient"}, "user_id": "18b9a79bb9d14fa595c67ee981f35158", "project_id": "1fcb9d1fbad0402197d275c63dc7df25", "request_id": "req-f7e044d7-546b-4eea-bccc-c4c2ebd4b7d4", "username": "admin"}, "target": {"id": "5cde627c-0b55-5726-918f-f1d94a0cf12c", "typeURI": "service/security/account/user"}, "project": "1fcb9d1fbad0402197d275c63dc7df25", "user": "72a8d0505eaa478684dfc0da4767db2f", "inherited_to_projects": false, "role": "a92a5e462003408c99ef427e404d6e51"}, "timestamp": "2025-09-30 04:01:53.361160"}}
```
Alert will trigger if a new role assignment is detected.

### CharmedOpenStackHighLoginFailureRate
**Example log line:**
In `/var/log/keystone/keystone.log`:
```
2025-09-30 11:49:47.660
{"message_id": "d3423246-4491-4283-b407-69d44cd4f114", "publisher_id": "identity.juju-644baf-yoga-ceph-6", "event_type": "identity.authenticate", "priority": "INFO", "payload": {"typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event", "eventType": "activity", "id": "597c63e3-f1cd-5488-a41b-135733c808d0", "eventTime": "2025-09-30T01:49:47.431826+0000", "action": "authenticate", "outcome": "failure", "observer": {"id": "4eade777dfc54e3a81a299b808ea4af6", "typeURI": "service/security"}, "initiator": {"id": "18b9a79bb9d14fa595c67ee981f35158", "typeURI": "service/security/account/user", "host": {"address": "10.149.54.84", "agent": "openstacksdk/0.61.0 keystoneauth1/4.4.0 python-requests/2.25.1 CPython/3.10.12"}, "request_id": "req-a0d99f5f-c103-4b80-b91f-76a7ec69cedd", "user_id": "18b9a79bb9d14fa595c67ee981f35158", "username": "admin"}, "target": {"id": "4161fa2b-2512-57a8-a9b6-1a1125eaa08c", "typeURI": "service/security/account/user"}}, "timestamp": "2025-09-30 01:49:47.606074"}}
```
Alert will trigger if the proportion of such lines with "outcome": "failure" exceeds a threshold in a period of time.

### CharmedOpenStackMultipleFailedLogins*
**Example log line:**
In `/var/log/keystone/keystone.log`:
```
2025-09-30 11:49:47.660
{"message_id": "d3423246-4491-4283-b407-69d44cd4f114", "publisher_id": "identity.juju-644baf-yoga-ceph-6", "event_type": "identity.authenticate", "priority": "INFO", "payload": {"typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event", "eventType": "activity", "id": "597c63e3-f1cd-5488-a41b-135733c808d0", "eventTime": "2025-09-30T01:49:47.431826+0000", "action": "authenticate", "outcome": "failure", "observer": {"id": "4eade777dfc54e3a81a299b808ea4af6", "typeURI": "service/security"}, "initiator": {"id": "18b9a79bb9d14fa595c67ee981f35158", "typeURI": "service/security/account/user", "host": {"address": "10.149.54.84", "agent": "openstacksdk/0.61.0 keystoneauth1/4.4.0 python-requests/2.25.1 CPython/3.10.12"}, "request_id": "req-a0d99f5f-c103-4b80-b91f-76a7ec69cedd", "user_id": "18b9a79bb9d14fa595c67ee981f35158", "username": "admin"}, "target": {"id": "4161fa2b-2512-57a8-a9b6-1a1125eaa08c", "typeURI": "service/security/account/user"}}, "timestamp": "2025-09-30 01:49:47.606074"}}
```
Alert will trigger if there are many such lines with "outcome": "failure" from the same username in a period of time.

---
