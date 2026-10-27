# Ubuntu Pro (`osConfiguration.ubuntuPro`)

## Intro

Attach VKS Ubuntu nodes to an Ubuntu Pro subscription and enable Pro services (ESM, Livepatch,
and so on) declaratively. You store the Pro token in a Secret and reference it from the
`osConfiguration.ubuntuPro` variable of the built-in ClusterClass. No custom ClusterClass is
needed.

## Requirements

- ClusterClass `builtin-generic-v3.3.0` or later.
- Ubuntu nodes: set `run.tanzu.vmware.com/resolve-os-image: os-name=ubuntu` on the controlPlane
  and on each machineDeployment.
- A valid Ubuntu Pro token from the [Ubuntu Pro dashboard](https://ubuntu.com/pro/dashboard).
- A Secret in the **same namespace as the Cluster** with a key named `token` (no trailing
  newline).
- Nodes must be able to reach Canonical's Pro endpoints, or a Landscape server in air-gapped
  environments.

## Why enable it

Ubuntu Pro adds extended security maintenance for the OS and application packages, kernel
Livepatch, and the FIPS-certified components that FIPS mode on Ubuntu relies on. It is the
supported route for long-term security patching on Ubuntu nodes.

## Limits

- Ubuntu nodes only. The variable is ignored on other OS types, so mixed-OS clusters are fine.
- Changes require a machine rollout.
- Pro is attached when the node boots, not baked into the image.
- Rotating the token: use a new Secret name and change `tokenSecretRef`. Updating the Secret
  contents alone is not documented to roll the nodes.
- Not supported together with AIDE (see `../aide`).

## Fields you can change

Variable path: `osConfiguration.ubuntuPro`. Scopes: cluster, controlPlane, workers (per
machineDeployment through `variables.overrides`).

| Field | Type | Required | Description |
|---|---|---|---|
| `tokenSecretRef` | string | yes | Name of the Secret holding the token under the key `token`. |
| `services` | list of strings | no | Pro services to enable, for example `esm-infra`, `esm-apps`, `livepatch`. |
| `settings` | list of `{key, value}` | no | Ubuntu Pro client (`ubuntu-advantage-tools`) settings. |

If `osConfiguration.systemProxy` is set, its HTTP and HTTPS values are also applied to the Pro
client proxy settings.
<!-- To verify: not stated in the public variable reference; check `sudo pro config show` on a node. -->

## Sample files

| File | What it does |
|---|---|
| `00-basic.yaml` | Token Secret plus a Cluster that attaches every Ubuntu node. |
| `01-per-pool.yaml` | Attaches only one worker pool, using its own token Secret. |
| `02-fips-and-proxy.yaml` | Pro combined with FIPS mode and a system proxy. |

Replace `<storage-class>`, `<your-ubuntu-pro-token>` and the Kubernetes version with values from
your environment.

## Verify

    kubectl get secret ubuntu-pro-token -n my-vsphere-namespace -o jsonpath='{.data.token}' | base64 -d | wc -c

On an Ubuntu node:

    sudo pro status
    sudo pro config show

## References

- [ClusterClass variable reference](https://developer.broadcom.com/xapis/vmware-vsphere-kubernetes-service/latest/variable-docs.html)
- [Ubuntu Pro documentation](https://ubuntu.com/pro/docs/understanding-pro-token/)
