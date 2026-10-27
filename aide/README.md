# AIDE file integrity monitoring (`osConfiguration.aide`)

## Intro

AIDE (Advanced Intrusion Detection Environment) builds a baseline of system files on each node
and periodically checks the filesystem against it, so unauthorized changes to the node OS are
detected. You turn it on with the `osConfiguration.aide` variable of the built-in ClusterClass.
No custom ClusterClass is needed.

## Requirements

- ClusterClass `builtin-generic-v3.8.0` or later.
- Kubernetes release 1.37 or later (admission rejects older releases when AIDE is enabled).
  <!-- To verify: not stated in the public variable reference; confirm on your VKS release. -->
- Linux nodes.

## Why enable it

AIDE is the file integrity monitoring control commonly required by DISA STIG and CIS node
hardening baselines. It is off by default because it adds bootstrap time and periodic check
load.

## Limits

- Linux only; Windows node pools are skipped.
- Changes require a machine rollout.
- Not supported together with Ubuntu Pro subscriptions (see `../ubuntupro`), because Pro updates
  packages in the background and causes drift.
- Editing monitored files directly on a node (for example `/etc` or `/usr/bin` over SSH) fails
  the integrity check. A node with unresolved drift also blocks later VKS-managed updates on
  that node until the drift is fixed.
- Optional third-party addons are not guaranteed to be noise-free. Validate in a lower
  environment before enabling broadly.

## Fields you can change

Variable path: `osConfiguration.aide`. Scopes: cluster, controlPlane, workers (per
machineDeployment through `variables.overrides`).

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `aide.enabled` | boolean | no | `false` | Enable AIDE on the node. |

## How it behaves on a node

1. During bootstrap the node installs AIDE, builds the baseline database and enables a daily
   check (`aide-check.timer`; cron on Ubuntu).
<!-- To verify: taint name, node condition names and file paths below are not in the public docs; confirm on a test cluster. -->
2. New nodes carry the taint `machine-agent.vks.vmware.com/bootstrap=pending:NoSchedule` until
   the baseline is ready. A node that stays tainted failed baseline creation.
3. Two conditions on the guest cluster Node report status: `AIDEBaselineReady` and
   `AIDEIntegrityCheckPassed` (`False` means drift was detected).
4. VKS-managed configuration changes (SSH banner, CAs, proxy) run a pre-check and then
   re-baseline automatically.
5. Reports are written to `/var/log/aide/aide.log`. Forward it with your log shipper.

## Sample files

| File | What it does |
|---|---|
| `00-enable-cluster-wide.yaml` | Enables AIDE on every Linux node in the cluster. |
| `01-per-pool.yaml` | Enables AIDE on the control plane and one worker pool only. |

Replace `<storage-class>` and the Kubernetes version with values from your environment.

## Verify

    kubectl --kubeconfig my-cluster.kubeconfig get nodes -o custom-columns=\
    NAME:.metadata.name,\
    BASELINE:.status.conditions[?\(@.type==\"AIDEBaselineReady\"\)].status,\
    INTEGRITY:.status.conditions[?\(@.type==\"AIDEIntegrityCheckPassed\"\)].status

On a node:

    sudo cat /var/run/machine-agent/aide-result.json
    sudo tail /var/log/aide/aide.log
    sudo systemctl status aide-check.timer aide-check.service
    sudo aide --check

## Remediation

- Unauthorized change: restore the files or replace the node, then
  `sudo systemctl reset-failed aide-check.service && sudo systemctl start aide-check.service`.
- Authorized change: apply it through a VKS-managed configuration path so the baseline is
  refreshed for you.
- Node stuck on the bootstrap taint: check free space on `/var` and `/var/log/machine-agent.log`.

## References

- [ClusterClass variable reference](https://developer.broadcom.com/xapis/vmware-vsphere-kubernetes-service/latest/variable-docs.html)
