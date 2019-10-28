# managed-cluster-config repository

This repo contains static configuration specific to a "managed" OpenShift Dedicated (OSD) cluster.

## How to use this repo

To add a new SelectorSyncSet:
1. Add your yaml manifest to the `deploy` dir
2. Add labels, based to which cloud provider this SelectorSyncSet should target:
```
  # Target GCP only
  labels:
    api.openshift.com/managed: "true"
    hive.openshift.io/cluster-platform: "gcp"

  # Target AWS only
  labels:
    api.openshift.com/managed: "true"
    hive.openshift.io/cluster-platform: "aws"

  # Target All clouds
  labels:
    api.openshift.com/managed: "true"
```
3. Run the `make` command.

# Building

## Dependencies

- oyaml: `pip install oyaml`

# Selector Sync Sets included in this repo

## Prometheus

A set of rules and alerts that SRE requires to ensure a cluster is functioning.  There are two categories of rules and alerts found here:

1. SRE specific, will never be part of OCP
2. Temporary addition until made part of OCP

## Prometheus and Alertmanager persistent storage

Persistent storage is configured using the configmap `cluster-monitoring-config`, which is read by the cluster-monitoring-operator to generate PersistentVolumeClaims and attach them to the Prometheus and Alertmanager pods.

## SRE Authorization

Instead of SRE having the `cluster-admin` role, a new ClusterRole, `osd-sre-admin`, is created with some permissions removed.  The ClusterRole can be regenerated in the `generate/sre-authorization` directory.  The role is granted to SRE via `osd-sre-admins` group.

To elevate privileges, SRE can add themselves to the group `osd-sre-cluster-admins`, which is bound to the ClusterRole `cluster-admin`.  When this group is created and managed by Hive, all users are wiped because the SelectorSyncSet will always have `users: null`.  Therefore, SRE will get elevated privileges for a limited time.

## Curated Operators

Initially OSD will support a subset of operators only.  These are managed by patching the OCP shipped OperatorSource CRs.  See `deploy/osd-curated-operators`.

NOTE that ClusterVersion is being patched to add overrides.  If other overrides are needed we'll have to tune how we do this patching.  It must be done along with the OperatorSource patching to ensure CVO doesn't revert the OperatorSource patching.

## Console Branding

Docs TBA.

## OAuth Templates

Docs TBA.

## Resource Quotas

Refer to [deploy/resource/quotas/README.md](deploy/resource/quotas/README.md).

## Image Pruning

Docs TBA.

## Dependencies

pyyaml

## Logging

Prepares the cluster for `elasticsearch` and `logging` operator installation and pre-configures curator to retain 2 days of indexes (1 day for operations).

To opt-in to logging, the customer must:
1. install the `logging` operator
2. install the `elasticsearch` operator
3. create `ClusterLogging` CR in `openshift-logging`


# Additional Scripts

There are additional scripts in this repo as a holding place for a better place or a better solution / process.

## Cluster Upgrade

Script `scripts/create-upgrade.sh` is used to upgrade all clusters managed by one Hive instance.  The script requires two inputs: the starting version and the target version.  If the target version is not available in the upgrade graph the upgrade will not be done.

To use the script:
1. kubectl/oc login into the desired Hive cluster
2. run the script, i.e. scripts/cluster-upgrade.sh 4.1.0 4.1.1
3. do #2 until there are no clusters listed as "progressing"

Note the script isn't a perfect solution.  It requires being run multiple times.  Whoever runs it must watch how long cluster upgrades are progressing.  If a cluster is taking a long time it's possible additional steps are needed in the cluster.  Usually this is some cluster operator is in a degraded state and needs fixing.
