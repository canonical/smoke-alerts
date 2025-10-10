# Centralized Alert Rule Repository for Smoke Detector

Contains "smoke-alarm" rules for these use cases:

- charmed microk8s
- canonical k8s
- sunbeam
- charmed openstack

This repository should be referenced by the [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s) charm to forward alert rules to Canonical Observability Stack (COS).

See audit logging setup guides for each use case:
- [Charmed OpenStack](charmed_openstack_setup.md)
- [Canonical K8s](canonical_k8s_setup.md)
