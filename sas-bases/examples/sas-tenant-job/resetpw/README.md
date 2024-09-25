---
category: multiTenant
tocprty: 7
---

# Generate Password Reset URL for Tenant SASProvider

## Overview

The tenant job can generate unique URLs that reset the SASProvider password of one or more tenants. The steps to execute the job are similar to those for onboarding tenants. However, the execution of the job only generates the reset URL and does no onboarding or service restarts.

## Prepare to Generate Reset URL

### Permissions

Elevated Kubernetes permissions are required to run the tenant job and perform the prerequisite steps.

### Copy the Tenant Job ResetPW Files

1. If you have any files in `$deploy/site-config/sas-tenant-job/resetpw/`, move them to a new sub-directory or delete them to ensure that you utilize the latest files in the next step.

2. Always copy the latest files from `$deploy/sas-bases/examples/sas-tenant-job/resetpw/*` to `$deploy/site-config/sas-tenant-job/resetpw/`.

3. Add write permission to all of the YAML files in the `$deploy/site-config/sas-tenant-job/resetpw/` directory.

### Verify Service Account Role for the Tenant Job

The tenant job requires the service account role so that it can make API calls. The role is created before tenants are added to the deployment. Verify that the role has not been dropped.

   ```
   kubectl -n {{ name-of-namespace }} get rolebinding sas-tenant-job
   ```

If the command returns a Not Found error, apply the service account role for the tenant job. The instructions for applying the role are located in `$deploy/sas-bases/examples/sas-tenant-job/README.md` (for Markdown format) or at `$deploy/sas-bases/docs/onboard_or_offboard.htm` (for HTML format).

## Generate the Reset Password URL

1. Ensure that you are in the new tenant resetpw directory, such as `$deploy/site-config/sas-tenant-job/resetpw/`.

2. Edit the job file `tenant-resetpw-job.yaml`.

 - Modify the job name `sas-tenant-resetpw-job-{{ JOB-TAG }}` by replacing `{{ JOB-TAG }}` with a distinguishing identifier. In the case of a single tenant, try adding the tenant identifier. In the case of multiple tenants, use a name that would help identify the group of tenants. The resulting job name must be unique in the namespace.

     Example:

         Job name for "acme" tenant:  sas-tenant-resetpw-job-acme

    **Note:**
    The job name must be unique or job creation will fail in step 6.

 - Replace `{{ SAS-TENANT-IDS }}` with the valid tenant ID/tenant IDs that will have their SASProvider passwords reset.

     Example:

        Single tenant ID: "acme"

        or

        Multiple tenant IDs: "acme, cyberdyne, intech"

        **Note:**
          Quotation marks and commas are required if you include multiple tenant IDs in one action.

3. Edit the file `kustomization.yaml` .
 - Replace the occurrence of `{{ VIYA-DEPLOYMENT-NAMESPACE }}` with the name of your SAS Viya platform namespace.
 - If your SAS Viya platform deployment disables Transport Layer Security (TLS), comment out the appropriate line:

   ```
   #- tenant-resetpw-job-tls-transformer.yaml
   ```

4. In the new resetpw directory, build the manifest file.

   **NOTE:** The examples in this step assume Kustomize version 4 is being used.  If you are using Kustomize version 3 then the `load-restrictor` option must be specified as `--load_restrictor=none`.

   ```
   kustomize build . -o resetpw.yaml --load-restrictor LoadRestrictionsNone
   ```

5. Get the full name of the `sas-image-pull-secrets` currently in your deployment.

   ```
   kubectl get secrets -n {{ name-of-namespace }} | grep sas-image-pull-secrets
   ```

   Then edit the resetpw.yaml file by replacing `sas-image-pull-secrets` with the full secret name you just retrieved.


6. Run the tenant job to generate the password reset link:

    ```
    kubectl create -f resetpw.yaml
    ```

    The job runs under the name provided in step 2 above.

7. Use the following commands to track the job using the name provided in step 2.

    To show details for a reset password job:

    ```
    kubectl -n {{ name-of-namespace }} describe job/{{ sas-tenant-resetpw-job-name }}
    ```

    To view the log of a specific job, first find its pod name:

    ```
    kubectl -n {{ name-of-namespace }} logs job/{{ sas-tenant-resetpw-job-name }}
    ```

### Reset the Password

After the successful completion of the job, locate the generated URL in the job log and use it to specify the new password. If multiple tenants were specified in the job, then a unique URL is generated for each tenant.

Use the following steps:

1. Get the password reset link from the log. This link expires 24 hours after the tenant is onboarded.

    ```
    kubectl -n {{ name-of-namespace }} logs job/{{ sas-tenant-resetpw-job-name }} | grep reset_password
    ```

    The link is a relative path in the form of `/SASLogon/reset_password?code=<resetCode>` where the resetCode appears as a random set of characters.

2. Paste the reset link into the address bar of a browser and add the tenantID and host information so that the address takes the form of a full URL:

    ```
    https://<tenantID>.<hostname>/SASLogon/reset_password?code=<resetCode>
    ```

   Go to the URL to reset the password.
