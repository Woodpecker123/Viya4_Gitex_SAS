---
category: SAS Event Stream Processing
tocprty: 4
---

# Configuring Access to Persistent Volumes for SAS Event Stream Processing Studio

## Overview

SAS Event Stream Processing requires a file-based PersistentVolume (PV) for running projects.

To configure SAS Event Stream Processing Studio to use persistent volumes
when running ESP projects with the ESP Operator, two changes are required:

* make a PersistentVolumeClaim (PVC) that creates a PersistentVolume (PV).
* set an environment variable with the name of the PVC

For deployments with a single tenant, the PVC is named `sas-event-stream-processing-studio-app`. In a multi-tenant environment, the PVC is named `sas-event-stream-processing-studio-app-{{ TENANT-NAME }}`.

The transformer adds the following environment variable: SAS_ESP_COMMON_KUBERNETES_DEFAULTS_PERSISTENTVOLUMECLAIM. The environment variable has a value of sas-event-stream-processing-studio-app.

After these changes are made, a PV is created from the PVC. The PV is
mounted from every SAS Event Stream Processing container. SAS Event Stream Processing containers are created when ESP projects are started from
SAS Event Stream Processing Studio.

The container mounts the PV from the directory `mnt/data`.

PVs are required for the following:

* to access files when you test or run projects
* to publish analytic store models from SAS Model Manager and Model Studio to a SAS Micro Analytic Service publishing destination
* to package projects, storing all project-related files under a single project directory

Therefore, SAS requires that you configure SAS Event Stream Processing to access persistent volumes.

## Prerequisites

* The storage must support ReadWriteMany access.
* Determine the STORAGE-CAPACITY required for ESP project XML, input and output streaming data files,
analytical models, and any other external files required by SAS Event Stream Processing.
* Make a note of the STORAGE-CLASS-NAME from the provider.
* For multi-tenant environments, a PVC is required for each tenant.

## Installation for a Single Tenant

1. Copy the files in the `$deploy/sas-bases/examples/sas-event-stream-processing-studio-app/pvc` directory to the `$deploy/site-config/sas-event-stream-processing-studio-app/pvc` directory. Create the destination directory if it does not exist.

2. The resources.yaml file in the `$deploy/site-config/sas-event-stream-processing-studio-app/pvc` directory has the parameters of the storage required in the PVC.
    * Replace {{ STORAGE-CAPACITY }} with the amount of storage required.
    * Replace {{ STORAGE-CLASS-NAME }} with the appropriate storage class from the cloud provider that supports the ReadWriteMany access mode.

3. Make the following changes to the base kustomization.yaml file in the $deploy directory.
    * Add site-config/sas-event-stream-processing-studio-app/pvc/resources.yaml to the resources block.
    * Add sas-bases/overlays/sas-event-stream-processing-studio-app/pvc/pvc-transformer.yaml to the transformers block.

The references should look like this:

 ```
 ...
 resources:
 ...
 - site-config/sas-event-stream-processing-studio-app/pvc/resources.yaml
 ...
 transformers:
 ...
 - sas-bases/overlays/sas-event-stream-processing-studio-app/pvc/pvc-transformer.yaml
 ...
 ```

After the base kustomization.yaml file is modified, deploy the software using
the commands described in [SAS Viya Platform Deployment Guide](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=titlepage.htm).

## Installation on a Multi-tenant System

**IMPORTANT:** Repeat these steps for each new tenant that will be onboarded.

1. Copy the files in the `$deploy/sas-bases/examples/sas-event-stream-processing-studio-app/pvc` directory to the `$deploy/site-config/sas-event-stream-processing-studio-app/pvc` directory. Create the destination directory if it does not exist.

2. For each tenant that you are onboarding, create a copy of resources-mt.yaml in the `$deploy/site-config/sas-event-stream-processing-studio-app/pvc` directory and replace "mt" in the file name with the tenant name.
For example: resources-acme.yaml, resources-cyberdyne.yaml, and resources-intech.yaml.

3. Each resources-*.yaml file in the `$deploy/site-config/sas-event-stream-processing-studio-app/pvc` directory has the parameters of the storage required in the PVC.
    * Replace {{ STORAGE-CAPACITY }} with the amount of storage required in each resources-*.yaml file.
    * Replace {{ STORAGE-CLASS-NAME }} with the appropriate storage class from the cloud provider that supports the ReadWriteMany access mode in each resources-*.yaml file.

4. Replace {{ TENANT-NAME }} in each resources-*.yaml file with the tenant name.

5. Make the following changes to the base kustomization.yaml file in the $deploy directory.
    * For each tenant, add site-config/sas-event-stream-processing-studio-app/pvc/resources-*.yaml to the resources block.
    * Add sas-bases/overlays/sas-event-stream-processing-studio-app/pvc/pvc-transformer.yaml to the transformers block.

The references should look like this:

 ```
 ...
 resources:
 ...
 - site-config/sas-event-stream-processing-studio-app/pvc/resources-acme.yaml
 - site-config/sas-event-stream-processing-studio-app/pvc/resources-cyberdyne.yaml
 - site-config/sas-event-stream-processing-studio-app/pvc/resources-intech.yaml
 ...
 transformers:
 ...
 - sas-bases/overlays/sas-event-stream-processing-studio-app/pvc/pvc-transformer.yaml
 ...
 ```

After the base kustomization.yaml file is modified, deploy the software using
the commands described in [SAS Viya Platform Deployment Guide](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=titlepage.htm).