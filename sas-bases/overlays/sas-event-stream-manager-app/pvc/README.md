---
category: SAS Event Stream Processing
tocprty: 5
---

# Configuring Access to Persistent Volumes for SAS Event Stream Manager

## Overview

SAS Event Stream Processing requires a file-based PersistentVolume (PV) for running projects.

To configure SAS Event Stream Processing Manager to use persistent volumes 
when running ESP projects with the ESP Operator, two changes are required:

* a PersistentVolumeClaim (PVC) is made to create a PersistentVolume (PV)
* an environment variable is set with the name of the PVC

The PVC is created by SAS Event Stream Processing Studio.

The transformer adds the following environment variable: SAS_ESP_COMMON_KUBERNETES_DEFAULTS_PERSISTENTVOLUMECLAIM

The environment variable has the following value: *sas-event-stream-processing-studio-app*

After these changes are made, the PersistentVolume (PV) is mounted from 
every ESP container created when starting an ESP project from 
SAS Event Stream Manager.

The container will mount the PV from the directory "/mnt/data".

*IMPORTANT* Persistent volumes are required for the following:

* to access files when you test or run projects
* to publish analytic store models from SAS Model Manager and Model Studio to a SAS Micro Analytic Service publishing destination
* to package projects, storing all project-related files under a single project directory

Therefore, SAS requires that you configure SAS Event Stream Processing to access persistent volumes.

## Prerequisites

Before proceeding, ensure that the sas-event-stream-processing-studio-app PVC has been created.

Consult the $deploy/sas-bases/examples/sas-event-stream-processing-studio-app/pvc/README.md file.

## Installation

In the base kustomization.yaml file in the $deploy directory, add 
sas-bases/overlays/sas-event-stream-manager-app/pvc/pvc-transformer.yaml to the
transformers block. The reference should look like this:

```
...
transformers:
...
- sas-bases/overlays/sas-event-stream-manager-app/pvc/pvc-transformer.yaml
...
```

After the base kustomization.yaml file is modified, deploy the software using 
the commands described in [SAS Viya Platform Deployment Guide](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=titlepage.htm).