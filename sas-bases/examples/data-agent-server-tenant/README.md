---
category: SAS Cloud Data Exchange
tocprty: 5
---

# Configure the Co-located SAS Data Agent Tenant CronJob

## Overview

This README file describes the configuration of the sas-data-agent-server-tenant CronJob used when onboarding or offboarding co-located SAS Data Agent tenants.

## Prerequisites

Before you start the deployment you should determine the OAuth secret that will be shared by the co-located SAS Data Agent tenant and any remote SAS Data Agents.

## Onboard a Tenant

Here are the basic steps to onboard a co-located SAS Data Agent tenant.

### Specify Options via the ConfigMap

1. Find the name of the ConfigMap:

   ```bash
   kubectl get configmap | grep sas-data-agent-server-tenant-config
   ```

2. Get the current ConfigMap options and save them to a file:

   ```bash
   kubectl get configmap sas-data-agent-server-tenant-config-\<hash from step 1\> -o yaml \> tda-config.yaml
   ```

3. Edit the tda-config.yaml file and change any values based on the descriptions in "ConfigMap Keys and Values" below:

   ```bash
   vi tda-config.yaml
   ```
   
4. Update the ConfigMap with your changes:

   ```bash
   kubectl apply -f tda-config.yaml
   ```

### Specify the OAuth Secret

The value for the OAuth secret corresponds to the co-located SAS Data Agent SAS_DA_OAUTH_SECRET kustomization option. It should match the value specified for the remote SAS Data Agent.

1. Get a base64-encoded version of the secret you want to share between the new co-located SAS Data Agent tenant and any remote SAS Data Agents:

   ```bash
   echo -n "mysecret" | base64
   ```

2. Get the current secret and save it to the tda-secrets.yaml file:

   ```bash
   kubectl get secret sas-data-agent-server-tenant-secrets -o yaml \> tda-secrets.yaml
   ```

3. Edit the tda-secrets.yaml file:

   ```bash
   vi tda-secrets.yaml
   ```

4. Update the value for the SAS_DA_TENANT_OAUTH_SECRET key with the base-64 encoded secret. Here is an example:
   
   ```bash
   SAS_DA_TENANT_OAUTH_SECRET: bXlzZWNyZXQ=
   ```

5. Save the new secret:

   ```bash
   kubectl apply -f tda-secrets.yaml
   ```

6. Create the CronJob to onboard the new tenant:

   ```bash
   kubectl create job --from=cronjob/sas-data-agent-server-tenant \<unique name\>
   ```

## Offboard a Tenant

Here are the basic steps to offboard a co-located SAS Data Agent tenant.

1. Find the name of the ConfigMap:

   ```bash
   kubectl get configmap | grep sas-data-agent-server-tenant-config
   ```
   
2. Get the current ConfigMap options and save them to a file:

   ```bash
   kubectl get configmap sas-data-agent-server-tenant-config-\<hash from above\> -o yaml \> tda-config.yaml
   ```

3. Edit the file and set SAS_DA_TENANTID to the tenant to be offboarded:

   ```bash
   vi tda-config.yaml
   ```
   
4. Update the ConfigMap with your changes:

   ```bash
   kubectl apply -f tda-config.yaml
   ```

5. Create the CronJob to Onboard the New Tenant:

   ```bash
   kubectl create job --from=cronjob/sas-data-agent-server-tenant \<unique name\>
   ```

## ConfigMap Keys and Values

The following keys and values can be specified to the sas-data-agent-server-tenant CronJob via the ConfigMap.

### SAS_DA_TENANTID

This option sets the co-located SAS Data Agent SAS_DA_TENANTID option.
Enter a string that specifies the assigned ID for the new co-located SAS Data Agent tenant. 
This is a *REQUIRED* option and must be specified.
**Note:**  If you are performing an update, you can specify * to indicate that all tenants should be updated.

### SAS_DA_TENANT_JOB_ACTION

Enter ONBOARD to create a new tenant, OFFBOARD to remove a tenant, or UPDATE to update a tenant.
**Note:** The co-located SAS Data Agent default provider tenant updates tenants automatically when
its pod is restarted, so users do not generally need to update unless they require manual updates.
The default is "ONBOARD" if not specified.

## Monitor the Onboard/Offboard job

The CronJob will run in a pod containing the \<unique name\> you specified when you created it.

1. Get the CronJob pod name:

   ```bash
   kubectl get pods | grep \<unique name\>
   ```

2. View the CronJob log:

   ```bash
   kubectl logs \<pod name from above\>
   ```
## Reduce RBAC Permissions for Single Tenant Deployments

The Kubernetes RBAC security properties for sas-data-agent-server-tenant and sas-data-agent-server-colocated include 'verbs: *' because they use the Kubernetes dynamic API to apply changes when onboarding, offboarding, and updating tenants.
Some customers may not allow this level of privilege when attempting to deploy Viya.  Customers who do not wish to deploy multiple SAS Data Agent tenants can reduce the RBAC security requirements by performing the following steps before deployment:    

1. Copy the `$deploy/sas-bases/examples/sas-data-agent-server-colocated/single-tenant-colocated-rbac-transformer.yaml` file to `$deploy/sas-config/sas-data-agent-server-colocated/single-tenant-colocated-rbac-transformer.yaml`.

2. Copy the `$deploy/sas-bases/examples/data-agent-server-tenant/single-tenant-cronjob-rbac-transformer.yaml` file to `$deploy/sas-config/data-agent-server-tenant/single-tenant-cronjob-rbac-transformer.yaml`.

3. Add a reference to each of these transformers to the end of the transformers block
of the base kustomization.yaml file (`$deploy/kustomization.yaml`). Here is an
example:

   ```yaml
   transformers:
   ...
   - site-config/sas-data-agent-server-colocated/single-tenant-colocated-rbac-transformer.yaml
   - site-config/data-agent-server-tenant/single-tenant-cronjob-rbac-transformer.yaml
   ```

