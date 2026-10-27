# ClusterResourceSets (CRS)

## Intro

A `ClusterResourceSet` automatically applies a set of Kubernetes resources (namespaces, quotas,
RBAC, package repositories, pull secrets) to every guest cluster whose labels match. You create
the CRS and its ConfigMaps or Secrets in the Supervisor namespace that holds the Cluster. No
custom ClusterClass is needed.

## Requirements

- Permission to create `clusterresourcesets.addons.cluster.x-k8s.io` in the vSphere namespace.
  This is often denied to non-admins. Check first:

      kubectl auth can-i create clusterresourcesets.addons.cluster.x-k8s.io -n my-vsphere-namespace
      kubectl api-resources | grep -i resourceset

- A Cluster carrying the label each CRS selects on (see `00-cluster-label.yaml`).
- For sample 02: a reachable package bundle. kapp-controller already runs in VKS guest clusters.

## Why use it

It gives every new cluster the same baseline (namespaces, quotas, RBAC, addons, credentials)
without per-cluster manual steps or a custom ClusterClass.

## Limits

- No ordering between resources inside a CRS; use separate CRSs when order matters.
- 1 MiB limit per ConfigMap or Secret.
- Deleting a CRS does not remove the resources it already applied to guest clusters.
- `strategy: ApplyOnce` applies once; `Reconcile` re-applies when the source changes.
- Secret-sourced CRS resources must use the Secret type `addons.cluster.x-k8s.io/resource-set`.

## Fields you can change

| Field | Description |
|---|---|
| `spec.clusterSelector.matchLabels` | Which Clusters receive the resources. |
| `spec.strategy` | `ApplyOnce` (default) or `Reconcile`. |
| `spec.resources[]` | `kind` (`ConfigMap` or `Secret`) and `name` of each source object. |
| ConfigMap/Secret `data` | Any YAML manifests, one per key, applied to the guest cluster. |

## Sample files

| File | What it does |
|---|---|
| `00-cluster-label.yaml` | Labels a Cluster so the CRSs below select it. |
| `01-baseline-resources.yaml` | Baseline namespace, ResourceQuota and RBAC in every labelled cluster. |
| `02-package-repo-and-install.yaml` | Two-stage CRS: Carvel PackageRepository, then a PackageInstall. |
| `03-image-pull-secret.yaml` | Secret-sourced CRS that delivers an image pull secret. |

Replace every `<placeholder>` first.

## Verify

    kubectl apply -f 00-cluster-label.yaml   # or just label the existing Cluster
    kubectl apply -f 01-baseline-resources.yaml
    kubectl get clusterresourceset -n my-vsphere-namespace
    kubectl get clusterresourcesetbinding -n my-vsphere-namespace my-cluster -o yaml
    kubectl --kubeconfig my-cluster.kubeconfig get ns platform

## References

- [Cluster API ClusterResourceSet](https://cluster-api.sigs.k8s.io/tasks/experimental-features/cluster-resource-set)
