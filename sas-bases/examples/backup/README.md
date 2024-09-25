---
category: backupRestore
tocprty: 1
---

# SAS Viya Platform Backup

The files in this directory are used to create a backup of the SAS Viya platform. You can
perform a one-time backup or you can schedule a regular backup of your
deployment. For information about performing backups and using these files, see
[SAS Viya Platform Administration: Backup and
Restore](https://documentation.sas.com/?cdcId=sasadmincdc&cdcVersion=default&docsetId=calbr&docsetTarget=titlepage.htm).

**Note:** Ensure that the version indicated by the version selector for the
document matches the version of your SAS Viya platform software.

## Copy Backup Data to sas-common-backup-data PersistentVolume

**Warning:** The sas-backup-copy-job is going to be deprecated in near future and it is being replaced with sas-backup-pv-copy-cleanup-job cronjob.

You can also use a Kubernetes job to copy backup data to the sas-common-backup-data PersistentVolume.

1. Copy the file sas-backup-copy-job.yaml in the `$deploy/sas-bases/examples/backup` directory to the `$deploy/site-config/backup` directory. Create the target directory, if it does not already exist.

2. Examine the contents of the `$deploy/site-config/backup` directory. If a kustomization.yaml file is present, ensure that it has only the following content. If it does not exist, create the kustomization.yaml file with the following content in that directory.

   ```yaml
   resources:
   ...
   - sas-backup-copy-job.yaml
   ...
   ```

3. Add site-config/backup to the resources block of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an example:

   ```yaml
   resources:
   ...
   - site-config/backup
   ...
   ```

4. Build and Apply the Manifest

   As an administrator with cluster permissions, apply the edited files to your deployment by performing the steps described in [Modify Existing Customizations in a Deployment](https://documentation.sas.com/?cdcId=sasadmincdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=n1f2q6pp0gjheqn1jl204vptrubs.htm).
