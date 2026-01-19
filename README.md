# Centralized Alert Rule Repository for Smoke Detector

Contains "smoke-alarm" rules for these use cases:

- charmed microk8s
- canonical k8s
- sunbeam
- charmed openstack
- landscape server

This repository should be referenced by the [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s) charm to forward alert rules to Canonical Observability Stack (COS).

See reference and audit logging setup guides for each use case:
- [Charmed OpenStack](docs/setup_guides/charmed_openstack_setup.md)
- [Canonical K8s](docs/setup_guides/canonical_k8s_setup.md)
- [Landscape Server](docs/setup_guides/landscape_server_references.md)

See all alert rules and example log references in [rules.md](docs/rules.md) and [references.md](docs/references.md).
