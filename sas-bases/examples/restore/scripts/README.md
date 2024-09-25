---
category: backupRestore
tocprty: 16
---

# Restore Scripts

## Overview

This README file contains information about the execution of scripts that are potentially required for restoring the SAS Viya Platform from a backup.

## Append the Execute Permissions to Scripts

To execute the scripts below, append the execute permission to both by running the following command.

   ```bash
   chmod +x ./sas-backup-pv-copy-cleanup.sh ./scale-up-cas.sh
   ```

## Clean Up CAS Persistent Volumes

Persistent volumes are used by the CAS server for restoring CAS data for each tenant. To clean up the CAS persistent volumes after the restore job has completed, execute the sas-backup-pv-copy-cleanup.sh bash script with three arguments: namespace, operation to perform, and a comma-separated list of tenants.

   ```bash
    ./sas-backup-pv-copy-cleanup.sh [namespace] [operation] "[tenants]"
   ```

   Example for a Multi-Tenant Deployment

   ```bash
   ./sas-backup-pv-copy-cleanup.sh viya04 remove "provider,acme,cyberdyne"
   ```

   Example for a Non-Multi-Tenant Deployment

   ```bash
   ./sas-backup-pv-copy-cleanup.sh viya04 remove "default"
   ```

**Note:** The tenant name for a non-multi-deployment can be either "default" or "provider".

## Copy Backup Data to sas-common-backup-data PersistentVolume

You can also use a Kubernetes job (sas-backup-pv-copy-cleanup-job) to copy backup data to the sas-common-backup-data PersistentVolume.

1. To create a copy job from the cronjob sas-backup-pv-copy-cleanup-job, execute the sas-backup-pv-copy-cleanup.sh script with three arguments: namespace, operation to perform, and a comma-separated list of tenants.

   ```bash
    ./sas-backup-pv-copy-cleanup.sh [namespace] [operation] "[tenants]"
   ```

   Example for a Multi-Tenant Deployment

   ```bash
    ./sas-backup-pv-copy-cleanup.sh viya04 copy "provider,acme,cyberdyne"
   ```

   Example for a Non-Multi-Tenant Deployment

   ```bash
   ./sas-backup-pv-copy-cleanup.sh viya04 copy "default"
   ```

**Note:** The tenant name for a non-multi deployment can be either "default" or "provider".

2. The script creates a copy job for each tenant that is included in the comma-separated list of tenants. Check for the sas-backup-pv-copy-cleanup-job pod that is created for each individual tenant.

   ```bash
   kubectl -n name-of-namespace get pod | grep -i sas-backup-pv-copy-cleanup
   ```
If you do not see the results you expect, see the console output of the sas-backup-pv-copy-cleanup.sh script.

The copy job pod mounts two persistent volumes (PVs) per tenant, per CAS instance. The 'sas-common-backup-data' PV is mounted at '/sasviyabackup' and the 'sas-cas-backup-data' PV is mounted at '/cas'.

## Scaling CAS Deployments

To scale up the CAS deployments that are used to restore CAS data for each tenant, execute the scale-up-cas.sh bash script with two arguments: namespace and a comma-separated list of tenants.

   ```bash
    ./scale-up-cas.sh [namespace] "[tenants]"
   ```

   Example for a Multi-Tenant Deployment

   ```bash
    ./scale-up-cas.sh viya04 "provider,acme,cyberdyne"
   ```

   Example for a Non-Multi-Tenant Deployment

   ```bash
    ./scale-up-cas.sh viya04 "default"
   ```

**Note:** The tenant name for a non-multi deployment can be either "default" or "provider".

Ensure that all the required sas-cas-controller pods are scaled up, especially if you have multiple CAS controllers.