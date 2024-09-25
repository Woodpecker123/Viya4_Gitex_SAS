---
category: Model Publish service
tocprty: 2
---

# Configure buildkit for SAS Model Publish Service

## Overview

[buildkit](https://github.com/moby/buildkit) is a tool to build container images from a Dockerfile without root privileges. The buildkit can load the build context from cloud storage or a local directory, and then push the built image to the container registry for a specific destination.

The Model Publish service uses the sas-model-publish-buildkit dedicated PersistentVolume Claim (PVC) as a workspace, which is shared with the buildkit container. When a user publishes a model to a container destination, sas-model-publish creates a temporary folder (publish-xxxxxxxx) on the volume (/models/buildkit/), which is then mounted from the sas-model-publish-buildkit PVC in the start-up process.

The publishing process generates the following content:

* Context (Dockerfile and model ZIP file) in the /models/buildkit/publish-xxxxxxxx directory.
* AWS configuration in the  /models/buildkit/publish-xxxxxxxx-aws directory, if AWS is the publishing destination type.
* Docker configuration in the /models/buildkit/publish-xxxxxxxx-docker directory (Required by all container destination types).

**Note:** The "xxxxxxxx" part of the folder names is a system-generated alphanumeric string and is 8 characters in length.

The Model Publish service then loads a pod template from the sas-model-publish-buildkit-job-config (as defined in podtemplate.yaml) and dynamically constructs a job specification. The job specification helps mount the directories in the buildkit container. The default pod template uses the official Buildkit image URL `moby/buildkit:v0.11.6-rootless`.

The BuildKit build is started after a batch job is executed. The Model Publish service checks the job status every 30 seconds. The job times out after 30 minutes, if it has not completed.

The Model Publish service deletes the job and the temporary directories after the job has completed successfully, completed with errors, or has timed out.

## Files

In order for the Model Publish service to successfully publish a model to a container destination, the user must prepare and adjust the following files that are located in the `$deploy/sas-bases/examples/sas-model-publish/buildkit` directory:

**storage.yaml**

  defines a PVC for the BuildKit workspace.

**podtemplate.yaml**

  defines a pod template for the batch job that launches the BuildKit build.
  
**buildkit-client-certs.yaml**

defines buildkit client certificates that launches the BuildKit build client job.

** sa.yaml

  defines the service account for running the BuildKit job.

The following file is located in the `$deploy/sas-bases/overlays/sas-model-publish/buildkit` directory and does not need to be modified:

**kaniko-transformer.yaml**

  adds the sas-model-publish-buildkit PVC to the sas-model-publish deployment object.

## Installation

1. Copy the files in the `$deploy/sas-bases/examples/sas-model-publish/buildkit` directory to the `$deploy/site-config/sas-model-publish/buildkit` directory. Create the destination directory, if it does not already exist.

   **Note:** If the destination directory already exists, [verify that the overlay](#verify-overlay-for-the-persistent-volume) has been applied. 
   If the output contains the /models/buildkit/ mount directory path, you do not need to take any further actions, unless you want to change the overlay parameters for the mounted directory.

2. Modify the parameters in the podtemplate.yaml file, if you need to implement customized requirements, such as the location of BuildKit image.

3. Modify the parameters in storage.yaml. For more information about PersistentVolume Claims (PVCs), see [Persistent Volume Claims on Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).

   * Replace {{ STORAGE-CAPACITY }} with the amount of storage required.
   * Replace {{ STORAGE-CLASS-NAME }} with the appropriate storage class from the cloud provider that supports ReadWriteMany access mode.

4. Make the following changes to the base kustomization.yaml file in the $deploy directory.

   * Add site-config/sas-model-publish/buildkit to the resources block.
   * Add sas-bases/overlays/sas-model-publish/buildkit/buildkit-transformer.yaml to the transformers block.

   Here is an example:
   
   ```yaml
   resources:
   - site-config/sas-model-publish/buildkit
   
   transformers:
   - sas-bases/overlays/sas-model-publish/buildkit/buildkit-transformer.yaml
   ```
     
5. Complete the deployment steps to apply the new settings. See [Deploy the Software](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=p127f6y30iimr6n17x2xe9vlt54q.htm) in _SAS Viya Platform: Deployment Guide_.

   **Note:** This overlay can be applied during the initial deployment of the SAS Viya platform or after the deployment of the SAS Viya platform.
   
   * If you are applying the overlay during the initial deployment of the SAS Viya platform, complete all the tasks in the README files that you want to use, then run `kustomize build` to create and apply the manifests. 
   * If the overlay is applied after the initial deployment of the SAS Viya platform, run `kustomize build` to create and apply the manifests.

## Verify Overlay for the Persistent Volume

1. Run the following command to verify whether the overlays have been applied:

   ```sh
   kubectl describe pod  <sas-model-publish-pod-name> -n <name-of-namespace>
   ```
   
2. Verify that the output contains the following mount directory paths:
    
   ```yaml
   Mounts:
     /models/buildkit
   ```

## Additional Resources

* [SAS Viya Platform: Deployment Guide](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=titlepage.htm)
* [Persistent Volume Claims on Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
* [Configuring Publishing Destinations](http://documentation.sas.com/?cdcId=mdlmgrcdc&cdcVersion=default&docsetId=mdlmgrag&docsetTarget=n0x0rvwqs9lvpun16sfdqoff4tsk.htm) in the _SAS Model Manager: Administrator's Guide_
