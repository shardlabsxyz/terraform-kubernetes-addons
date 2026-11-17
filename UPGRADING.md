# Dependency upgrade notes

## November 2026 dependency updates

This update changes the following dependency versions:

| Dependency | Previous | Updated |
| --- | --- | --- |
| `alekc/kubectl` provider (all modules) | `~> 2.0` | `>= 3.0.1, < 4.0` |
| `hashicorp/kubernetes` provider (AWS and Talos) | `< 3.4` | `< 3.6` |
| kube-prometheus-stack chart | `92.22.0` | `93.3.1` |
| Traefik chart | `41.7.1` | `42.4.1` |

The Generic module already allows Kubernetes provider 3.5 through its `~> 3.0` constraint. Terraform and OpenTofu runtime requirements are unchanged.

## Providers

Run `terraform init -upgrade` (or `tofu init -upgrade`) in the consuming root module and review its plan before applying. Commit the root module's updated provider lockfile with the upgrade.

The kubectl provider migrates existing v2 state automatically. Version 3 removes `yaml_incluster` and `live_manifest_incluster`, changes `kubectl_server_version.triggers` to `map(string)`, and requires `server_side_apply = true` when `force_conflicts = true`. This repository does not use the removed attributes or server-version triggers, and its manifest resource already enables server-side apply. The minimum is 3.0.1 to include the initial v3 drift fixes. See the [v3 upgrade notes](https://github.com/alekc/terraform-provider-kubectl/releases/tag/v3.0.0) and [3.0.1 fixes](https://github.com/alekc/terraform-provider-kubectl/releases/tag/v3.0.1).

Kubernetes provider 3.4 and 3.5 tighten validation for several v1 RBAC resources, including null metadata entries and simultaneous `name`/`generate_name` values. These resources are not declared by this repository, but the selected provider is shared with the consuming root module. Review the [provider changelog](https://github.com/hashicorp/terraform-provider-kubernetes/blob/v3.5.0/CHANGELOG.md) for other resources in that root module.

## Helm charts

Chart entries are shared dependency metadata; these changes affect installations that use the updated entries. Custom Helm values and existing clusters require their own upgrade plan.

- **Traefik 42** requires Kubernetes **1.30 or newer**. Traefik Hub now limits Secret write permissions to its release namespace and introduces a Secret-type admission policy when `hub.token` is configured. Transparency-log witness settings must include witnesses and a threshold of at least one. Review the [Traefik 42 upgrade notes](https://github.com/traefik/traefik-helm-chart/releases/tag/v42.0.0), especially when using Hub or custom RBAC.
- **kube-prometheus-stack 93** upgrades Prometheus Operator and changes its RBAC. Upgrade the Prometheus Operator CRDs using the chart's supported CRD upgrade job or the upstream manual procedure before upgrading existing installations. Follow the [chart upgrade guide](https://github.com/prometheus-community/helm-charts/blob/kube-prometheus-stack-93.3.1/charts/kube-prometheus-stack/UPGRADE.md#from-92x-to-93x).

Static validation and chart rendering do not verify an upgrade against existing cluster state. Retain the previous configuration, lockfile, and a state backup before applying provider migrations; reverting the version constraints alone does not undo a state migration.
