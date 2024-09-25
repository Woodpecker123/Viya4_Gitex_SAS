---
category: auditing
tocprty: 10
---

# Migrate SAS Audit Archived Data from SAS Viya 4 To SAS Viya 4

## Overview

Archived data from the audit process is stored in a persistent volume (PV) on both single-tenant environments and multi-tenant environments. In a multi-tenant environment, separate folders with the tenant name are created for each tenant in the same PV and data is organized by date. Audit and activity data are stored separately.

Audit service PVC data does not participate in the SAS Viya platform backup and restore procedure. Therefore it contains archived data that is never restored to the SAS Viya platform system. As a result, when audit archiving is performed, SAS recommends that the cluster administrator take a backup of the audit archive data and keep that data at a secure location.

## Prerequisites

1. The audit service should be running on both source and target environments and should have the PV attached.

2. To perform some elements of this task, you must have elevated Kubernetes permissions.

3. You should follow the steps described in [Hardware and Resource Requirements](https://go.documentation.sas.com/doc/en/itopscdc/default/itopssr/n0ampbltwqgkjkn1j3qogztsbbu0.htm) page, especially   section [Persistent Storage Volumes, PersistentVolumeClaims, and Storage Classes](https://go.documentation.sas.com/doc/en/itopscdc/default/itopssr/n0ampbltwqgkjkn1j3qogztsbbu0.htm#n1egh9hqndi6lin13w58nozc7vco).

4. The tenant names should be same in the source and target environments. If any source environment tenant does not exist on the target environment, then purge functionality will not work for that tenant on the target environment.

### Best Practices for Performing Manual Backup

1. Take frequent backups of audit archived data during off-peak hours. The frequency of backups should be determined by the frequency of archiving defined by your organization.

2. The PV that contains archived data is part of the same cluster as the environment. Therefore, SAS recommends that you routine copy archived data to storage outside of the cluster, such as NFS, in case the PV or entire cluster fails.

3. The time required to copy the archived audit data contents varies based on the size of the data, the disk I/O rate of the system, and the type of file system that you are using.

## Migrate Archived Audit Data from Source SAS Viya 4 Environment to Target SAS Viya 4 Environment

The following steps use a generic method (tar and kubectl exec) and an Audit service pod to copy data between environments. The steps in this generic method are not specific to any one cloud provider. You might follow a slightly different set of steps depending on what type of storage you are using for your data.

1. Log in to the cluster where you want to keep a backup of the archived data temporarily. You must have root level permissions to copy the archived data.

2. Determine the temporary location of the data that you wish to copy to and from.

3. Export or set the source machine kubeconfig file and then get source audit pod name.

   ```yaml
   export KUBECONFIG=<source-machine-kubeconfig>
   kubectl get pods -n <name-of-namespace> | grep sas-audit
   ```

4. Copy audit archived data from source machine to temporary location

    - To copy all the data from a single-tenant or multi-tenant deployment:
      
      ```yaml
      kubectl -n <name-of-namespace> exec <source-audit-pod-name> -- tar cpf - -C /archive . | tar xf - -C <temp-folder-path>
      ```

      Here is an example:

      ```yaml
      kubectl -n sourceTenant exec sas-audit-58cccfb4f7-pd870 -- tar cpf - -C /archive . | tar xf - -C /opt/tmpDir
      ```

    - To copy individual tenant data in multi-tenant deployments:

      ```yaml
      kubectl -n <name-of-namespace> exec <source-audit-pod-name> -- tar cpf - -C /archive/<storage-local-destination>/<tenant-name> . | tar xf - -C <temp-folder-path>
      ```

      Here is an example:

      ```yaml
      kubectl -n sourceTenant exec sas-audit-7648bcc97d-f7gwv -- tar cpf - -C /archive/auditData/acme . | tar xf - -C /opt/tmpDir
      ```

**Note:** `temp-folder-path` is the location that is being used to keep the data temporarily.
The `storage-local-destination` value should be taken from the ‘sas.audit.archive.system.storage.local.destination’ Audit service configuration property of the source environment. The default value for this property is ‘/auditData’. The value is configurable so it is possible that you might have changed it from the default. `tenant-name` is the name of the tenant being migrated. Each tenant has a folder created with the tenant name to store data separately in single Audit PV.

5. Export or set target machine kubeconfig on same setup and get target audit pod name.

   ```yaml
   export KUBECONFIG=<target-machine-kubeconfig>
   kubectl get pods -n <name-of-namespace> | grep sas-audit
   ```

6. Migrate the audit archived data from the temporary location to the target machine PV.
    - To migrate all the data from a single-tenant or multi-tenant deployment:

    ```yaml
    tar cpf - -C <temp-folder-path> * | kubectl -n <name-of-namespace> exec -i <target-audit-pod-name> -- tar xf - -C /archive
    ```

    Here is an example:

    ```yaml
    tar cpf - -C /opt/tmpDir * | kubectl -n targetTenant exec -i sas-audit-555c58c44f-ssjx7 -- tar xf - -C /archive
    ```

    - To migrate individual tenant data from multi-tenant deployments:

    ```yaml
    tar cpf - -C <temp-folder-path> * | kubectl -n <name-of-namespace> exec -i <target-audit-pod-name> -- tar xf - -C /archive/<storage-local-destination>/<tenant-name>
    ```

    Here is an example:
    
    ```yaml
    tar cpf - -C /opt/tmpDir * | kubectl -n targetTenant exec -i sas-audit-59555d895-kzpcg -- tar xf - -C /archive/auditData/acme
    ```

**Note:** The /archive/\<storage-local-destination>/\<tenant-name> folder location should already be present on the target PV. The `temp-folder-path` is the location where archived data is kept temporarily. The `storage-local-destination` value should be taken from the ‘sas.audit.archive.system.storage.local.destination’ Audit service configuration property of the target environment. The default value for this property is ‘/auditData’. The value is configurable so it is possible that you might have changed it from the default.
`tenant-name` is the name of tenant whose migration is being done solely. Each tenant has folder created with tenant name to store their data separately in single Audit PV.
