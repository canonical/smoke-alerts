# Centralized Alert Rule Repository for Smoke Detector

Contains "smoke-alarm" rules for these use cases:

- charmed microk8s
- canonical k8s
- sunbeam
- charmed openstack
- landscape server
- charmed kubeflow

This repository should be referenced by the [cos-configuration-k8s](https://charmhub.io/cos-configuration-k8s) charm to forward alert rules to Canonical Observability Stack (COS).

See audit logging and other setup guides for each use case:
- [Charmed OpenStack](docs/setup_guides/charmed_openstack_setup.md)
- [Canonical K8s](docs/setup_guides/canonical_k8s_setup.md)
- [Landscape Server](docs/setup_guides/landscape_server_references.md)
- [Charmed Kubeflow](docs/setup_guides/charmed_kubeflow_setup.md)

See all alert rules and example log references in [rules.md](docs/rules.md) and [references.md](docs/references.md).
