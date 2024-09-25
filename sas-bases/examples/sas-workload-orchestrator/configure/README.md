---
category: SAS Workload Orchestrator Service
tocprty: 1
---

# Configuration Settings for SAS Workload Orchestrator Service

## Overview

The SAS Workload Orchestrator Service is used to manage workload started on
demand through the launcher service. The service has manager pods in a
stateful set and server pods in a daemon set.

This README file describes the changes that can be made to the SAS Workload
Orchestrator Service settings for pod resource requirements and for
user-defined resource scripts.

### Pod Resource Requests and Limits

Kubernetes pods have resource requests and limits for CPU and memory.

Manager pods handle all the REST API calls and manage all of the processing
of host, job, and queue information. The more jobs you process at the same time,
the more memory and cores you should assign to the stateful set pods.
For manager pods, the current default resource request and limit for CPU and
memory is 1 core and 4GB of memory.

Server pods interact with Kubernetes to manage the resources and jobs running
on a particular node. Their memory and core requirements depend on how jobs are
allowed to concurrently run on a node and how many pods not started by the
SAS Workload Orchestrator Service are also running on a node.
For server pods, the current default resource request and limit for CPU and
memory is 0.1 core and 250MB of memory.

Generally, manager pods use more resources than daemon pods with the
resource request amount equalling the limit amount.

### Pod User-Defined Script Volume

SAS Workload Orchestrator allows user-defined resources to be used
for scheduling. User-defined resources can be a specified value or can
be a value returned by executing a script.

Manager pods handle the running of user-defined resource scripts for
resources that affect the scheduling on a global scale. An example of
a global resource would be the number of licenses across all pods started
by SAS Workload Orchestrator.

Server pods also handle the running of user-defined resource scripts for
resources that reflect values about an individual node that a pod would run on.
An example of a host resource could be number of GPUs on a host (for the
case of a static resource) or the amount of disk space left on a mount (for
the case of a dynamic resource).

In order to set these values, SAS Workload Orchestrator looks for a script
in a volume mount named "/scripts". To place a script in that directory,
the script must be placed in a volume and that volume specified in the
stateful set or daemon set definition as a volume with the name 'scripts'.

## Installation

Based on the following descriptions of available example files, determine if you
want to use any example file in your deployment. If so, copy the example
file and place it in your site-config directory.

The example files described in this README file are located at
'/$deploy/sas-bases/examples/sas-workload-orchestrator/configure'.

### StatefulSet Pods Requests and Limits

The values for memory and CPU resources for the SAS Workload Orchestrator Service manager pods
are specified in `sas-workload-orchestrator-statefulset-resources.yaml`.

To update the defaults, replace the `{{ MEM-REQUIRED }}` and `{{ CPU-REQUIRED }}` variables
with the values you want to use.

**Note:** It is important that the values for the requests and limits
be identical to get Guaranteed Quality of Service for the SAS Workload Orchestrator Service pods.

Here is an example:

```yaml
  - op: replace
    path: /spec/template/spec/containers/0/resources/requests/memory
    value: 6Gi
  - op: replace
    path: /spec/template/spec/containers/0/resources/limits/memory
    value: 6Gi
  - op: replace
    path: /spec/template/spec/containers/0/resources/requests/cpu
    value: "2"
  - op: replace
    path: /spec/template/spec/containers/0/resources/limits/cpu
    value: "2"
```

**Note:** For details on the value syntax used in the code, see
[Resource units in Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

After you have edited the file, add a reference to it to the transformers
block of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an example:

```yaml
transformers:
...
- site-config/sas-workload-orchestrator/configure/sas-workload-orchestrator-statefulset-resources.yaml
```

### DaemonSet Pods Requests and Limits

The values for memory and CPU resources for the SAS Workload Orchestrator Service server pods
are specified in `sas-workload-orchestrator-daemonset-resources.yaml`.

To update the defaults, replace the `{{ MEM-REQUIRED }}` and `{{ CPU-REQUIRED }}` variables
with the values you want to use.

**Note:** It is important that the values for the requests and limits
be identical to get Guaranteed Quality of Service for the SAS Workload Orchestrator Service pods.

Here is an example:

```yaml
  - op: replace
    path: /spec/template/spec/containers/0/resources/requests/memory
    value: 4Gi
  - op: replace
    path: /spec/template/spec/containers/0/resources/limits/memory
    value: 4Gi
  - op: replace
    path: /spec/template/spec/containers/0/resources/requests/cpu
    value: "1500m"
  - op: replace
    path: /spec/template/spec/containers/0/resources/limits/cpu
    value: "1500m"
```

**Note:** For details on the value syntax used in the code, see
[Resource units in Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes)

After you have edited the file, add a reference to it to the transformers
block of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an example:

```yaml
transformers:
...
- site-config/sas-workload-orchestrator/configure/sas-workload-orchestrator-daemonset-resources.yaml
```

### User-Defined Scripts Volume for Manager Pods

The following example mounts an NFS volume as the 'scripts' volume. It is the
example found in `sas-workload-orchestrator-global-user-defined-resources-script-storage.yaml`.

To update the volume, replace the `{{ NFS-SERVER-ADDR }}` variable with the fully-qualified
domain name of the server and replace the `{{ NFS-SERVER-PATH }}` variable with the path to
the volume on the server. Here is an example:

```yaml
  - op: replace
    path: /spec/template/spec/volumes/0
    value:
      name: scripts
      nfs:
        path: /path/to/my/scripts
        server: my.nfs.server.mydomain.com
```

Alternately, you could use any other type of volume Kubernetes supports.

The following example updates the volume to use a PersistentVolumeClaim
instead of an NFS mount. This assumes the PVC has already been defined and created.

```yaml
  - op: replace
    path: /spec/template/spec/volumes/0
    value:
      name: scripts
      persistentVolumeClaim:
        claimName: my-pvc-name
        readOnly: true
```

**Note:** For details on the value syntax used specifying volumes, see
[Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).

After you have edited the file, add a reference to it to the transformers
block of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an example:

```yaml
transformers:
...
- site-config/sas-workload-orchestrator/configure/sas-workload-orchestrator-global-user-defined-resources-script-storage.yaml
```

### DaemonSet Pods User-Defined Script Volume

The following example mounts an NFS volume as the 'scripts' volume. It is the
example found in `sas-workload-orchestrator-host-user-defined-resources-script-storage.yaml`.

To update the volume, replace the `{{ NFS-SERVER-ADDR }}` variable with the fully-qualified
domain name of the server and replace the `{{ NFS-SERVER-PATH }}` variable with the path to
the volume on the server. Here is an example:

```yaml
  - op: replace
    path: /spec/template/spec/volumes/0
    value:
      name: scripts
      nfs:
        path: /path/to/my/scripts
        server: my.nfs.server.mydomain.com
```

Alternately, you could use any other type of volume Kubernetes supports.

The following example updates the volume to use a PersistentVolumeClaim
instead of an NFS mount. This assumes the PVC has already been defined and created.

```yaml
  - op: replace
    path: /spec/template/spec/volumes/0
    value:
      name: scripts
      persistentVolumeClaim:
        claimName: my-pvc-name
        readOnly: true
```

**Note:** For details on the value syntax used specifying volumes, see
[Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).

After you have edited the file, add a reference to it to the transformers
block of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an example:

```yaml
transformers:
...
- site-config/sas-workload-orchestrator/configure/sas-workload-orchestrator-host-user-defined-resources-script-storage.yaml
```