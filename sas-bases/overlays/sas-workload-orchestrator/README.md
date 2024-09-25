---
category: SAS Workload Orchestrator Service
tocprty: 3
---

# Cluster Privileges for SAS Workload Orchestrator Service

## Overview

SAS Workload Orchestrator Service is an advanced scheduler that integrates with SAS Launcher. Adding this overlay allows
the service account to retrieve a cluster node's 'allocatable' amount of cores and memory, schedule jobs across
multiple namespaces, and read node labels to sort them into host groups. SAS recommends using this service so that your
deployment will run optimally.

## Instructions

### Enable the Cluster Role

The ClusterRole and ClusterRoleBinding are enabled by adding the file to the resources block of the base kustomization.yaml file
(`$deploy/kustomization.yaml`). Here is an example:

```yaml
resources:
...
- sas-bases/overlays/sas-workload-orchestrator
```

### Disable the Cluster Role

To disable the ClusterRole and ClusterRoleBinding:

1. Remove `sas-bases/overlays/sas-workload-orchestrator` from the resources block of the
base kustomization.yaml file (`$deploy/kustomization.yaml`). This also ensures that the
ClusterRole option will not be applied in future Kustomize builds.

2. Perform the following command to remove the ClusterRoleBinding from the namespace:

```
kubectl delete clusterrolebinding sas-workload-orchestrator-<your namespace>
```

3. Perform the following command to remove the ClusterRole from the cluster.

```
kubectl delete clusterrole sas-workload-orchestrator
```

## Build

After you configure Kustomize, continue your SAS Viya platform deployment as documented.