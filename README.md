# VKS advanced examples

Worked examples for configuring vSphere Kubernetes Service (VKS) clusters with the built-in
ClusterClass variables. Every folder follows the same layout: a `README.md` (Intro,
Requirements, Why enable it, Limits, Fields you can change, Sample files, Verify, References)
plus numbered sample manifests.

| Folder | Variable / feature | Minimum ClusterClass |
|---|---|---|
| [`tuned/`](tuned) | `osConfiguration.tuned` (TuneD kernel and OS tuning) | `builtin-generic-v3.6.0` |
| [`ubuntupro/`](ubuntupro) | `osConfiguration.ubuntuPro` (Ubuntu Pro attach) | `builtin-generic-v3.3.0` |
| [`aide/`](aide) | `osConfiguration.aide` (file integrity monitoring) | `builtin-generic-v3.8.0` |
| [`clusterresourcesets/`](clusterresourcesets) | ClusterResourceSet bootstrap resources | any |

## Conventions

- Cluster name `my-cluster`, vSphere namespace `my-vsphere-namespace`.
- `<angle-bracket>` values are placeholders you must replace.
- Cluster samples are complete `cluster.x-k8s.io/v1beta2` manifests; the lines that matter for
  the feature are marked `# <--`.
- Samples are numbered (`00-`, `01-`, ...) from simplest to most involved.
- The samples are examples; test them in a non-production environment first.
- Reference documentation: the
  [ClusterClass variable reference](https://developer.broadcom.com/xapis/vmware-vsphere-kubernetes-service/latest/variable-docs.html).
