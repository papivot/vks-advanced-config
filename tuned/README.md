# TuneD profiles (`osConfiguration.tuned`)

## Intro

Apply Linux kernel and OS tuning (sysctls, sysfs, boot arguments) to VKS nodes declaratively
with [TuneD](https://tuned-project.org/) profiles. You create `TuneDProfile` custom resources
in the Supervisor namespace and reference them from the `osConfiguration.tuned` variable of the
built-in ClusterClass. No custom ClusterClass is needed.

## Requirements

- ClusterClass `builtin-generic-v3.6.0` or later.
- Kubernetes release v1.34.0 or later.
- Linux nodes (Photon or Ubuntu).
- `TuneDProfile` CRs in the same namespace as the Cluster (the built-in profile
  `builtin-vks-v3.6.0` lives in `vmware-system-vks-public` and is referenced by name only).

## Why enable it

Workloads such as databases, caches and high-density platforms recommend kernel settings the
default node image does not apply (`vm.max_map_count`, `vm.swappiness`, hugepages, conntrack
sizing, and so on). TuneD is the supported way to set node-wide sysctls, and Broadcom recommends
it over enabling kubelet `unsafe-sysctls`, because most sysctls are not namespaced anyway.

## Limits

- Linux only; Windows nodes are not supported.
- Changes require a machine rollout.
- `TuneDProfile` CRs are immutable. Create a new CR (for example `-v2`) and repoint the Cluster.
- Admission rejects a `profileRef` to a CR that does not exist, and a profile that is injected
  (listed under `profiles`) but not listed under `active`.
- Boot-argument changes (hugepages, IOMMU) need a reboot.
- Incorrect values can make nodes unhealthy (for example a very low `fs.file-max`). Test on a
  dedicated pool first.
- Known issue: on VKS 3.6, changing a profile during heavy container churn can crash `containerd`
  and stall the rollout. See
  [KB 446653](https://knowledge.broadcom.com/external/article/446653/vks-36-workload-cluster-rollout-stuck-in.html)
  for the workaround.

## Fields you can change

Variable path: `osConfiguration.tuned`. Scopes: cluster, controlPlane, workers (per
machineDeployment through `variables.overrides`).

| Field | Type | Required | Description |
|---|---|---|---|
| `tuned.active` | list of strings | yes (min 1) | Profile names to activate on the node, applied in order. |
| `tuned.profiles` | map | no | Key is the profile name on the node. |
| `tuned.profiles.<name>.profileRef.name` | string | yes | Name of the `TuneDProfile` CR. |

What a profile (`TuneDProfile.spec.content`, INI format) can express: node-wide sysctls
(`[sysctl]`), sysfs and transparent hugepage settings (`[sysfs]`, `[vm]`), kernel boot arguments
(`[bootloader]`), any other TuneD plugin, and `include=` of stock profiles such as
`throughput-performance`, `latency-performance` and `virtual-guest`.

What it does not cover: kubelet flags, containerd configuration, or installing packages.

## Sample files

| File | What it does |
|---|---|
| `00-builtin-profile.yaml` | Cluster-wide activation of the built-in `builtin-vks-v3.6.0` profile. |
| `01-custom-db-profile.yaml` | Custom database profile applied to one worker pool. |
| `02-per-pool-profiles.yaml` | Redis and serverless profiles on two separate pools. |
| `03-layered-hugepages.yaml` | Built-in baseline layered with a custom hugepages/IOMMU profile. |

Replace `<storage-class>` and the Kubernetes version with values from your environment.

## Verify

    kubectl get tunedprofile -n my-vsphere-namespace
    kubectl get tunedprofile builtin-vks-v3.6.0 -n vmware-system-vks-public -o yaml

OS-level settings are not visible through the Kubernetes API. On a node:

    sudo tuned-adm active
    sudo tuned-adm verify
    sudo tail /var/log/tuned/tuned.log

## References

- [ClusterClass variable reference](https://developer.broadcom.com/xapis/vmware-vsphere-kubernetes-service/latest/variable-docs.html)
- [Unlocking Database Performance on Kubernetes with TuneDProfile in VKS 3.6](https://blogs.vmware.com/cloud-foundation/2026/03/20/tunedprofile-vks-3-6-os-level-tuning-kubernetes/)
