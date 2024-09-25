---
category: SAS Cloud Data Exchange
tocprty: 1
---

# Configure a Co-located SAS Data Agent

## Overview

The directory `$deploy/sas-bases/examples/sas-data-agent-server-colocated` contains files to customize your SAS Viya platform deployment for 
a co-located SAS Data Agent. This README describes the steps necessary 
to make these files available to your SAS Viya platform deployment. It also describes 
how to set required environment variables to point to these files. 

**Note:** If you make changes to these files after the initial deployment,
you must restart the co-located SAS Data Agent.

## Prerequisites

Before you start the deployment you should determine the OAUTH secret that will be used by co-located SAS Data Agent and any remote SAS Data Agents.

You should also create a subdirectory within `$deploy/site-config` to store your co-located SAS Data Agent configurations. This README uses a user-created subdirectory called 
`$deploy/site-config/sas-data-agent-server-colocated`. For more information, refer to the ["Directory Structure" section of the "Pre-installation
Tasks" Deployment Guide](https://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=p1goxvcgpb7jxhn1n85ki73mdxc8.htm&locale=en).

## Installation

The base kustomization.yaml file (`$deploy/kustomization.yaml`) provides configuration properties for the customization process.
The co-located SAS Data Agent requires specific customizations in order to communicate with remote SAS Data Agents and configure server options. Copy the example `sas-data-agent-server-colocated-config.properties` and `sas-data-agent-server-colocated-secret.properties` files from `$deploy/sas-bases/examples/sas-data-agent-server-colocated` to `$deploy/site-config/sas-data-agent-server-colocated`.

## Configuration

**Note:** The default values listed in the descriptions that follow should be suitable for most users.

### Configure the OAuth Secret

#### SAS_DA_OAUTH_SECRET

The sas-data-agent-server-colocated-secret.properties file contains configuration properties for the OAUTH secret. The OAUTH secret value is required and must be specified in order to communicate with a remote SAS Data Agent. There is no default value for the OAUTH secret.

**Note:** The following example is for illustration only and should not be used.

Enter a string value for the OAUTH secret that will be shared with the remote SAS Data Agent. Here is an example:

```bash
SAS_DA_OAUTH_SECRET=MyS3cr3t
```

### Configure Logging

The `sas-data-agent-server-colocated-config.properties` file contains configuration properties for logging.

#### SAS_DA_DEBUG_LOGTYPE

Enter a string value to set the level of additional logging.

   * `SAS_DA_DEBUG_LOGTYPE=TRACEALL` enables trace level for all log items.
   * `SAS_DA_DEBUG_LOGTYPE=TRACE` enables trace level for most log items.
   * `SAS_DA_DEBUG_LOGTYPE=PERFORMANCE` enables tracce/debug level items for performance debugging.
   * `SAS_DA_DEBUG_LOGTYPE=PREFETCH` enables trace/debug level items for prefetch debugging.
   * `SAS_DA_DEBUG_LOGTYPE=None` disables additional tracing.

If no value is specified, the default of None is used.

Here is an example:

```bash
SAS_DA_DEBUG_LOGTYPE=None
```

### Configure Filesystem Access

The `sas-data-agent-server-colocated-config.properties` file contains configuration properties that restrict drivers from accessing the container filesystem. By default, drivers can only access the directory tree `/data` which must be mounted on the co-located SAS Data Agent container.

#### SAS_DA_RESTRICT_CONTENT_ROOT

When set to TRUE, the file access drivers can only access the directory structure specified by SAS_DA_CONTENT_ROOT.

When set to FALSE, the file access drivers can access any directories accessible from within the co-located SAS Data Agent container.

If no value is specified, the default of TRUE is used.

```bash
SAS_DA_RESTRICT_CONTENT_ROOT=None
```

#### SAS_DA_CONTENT_ROOT

Enter a string value to specify the directory tree that file access drivers are allowed to access. This value is ignored if SAS_DA_RESTRICT_CONTENT_ROOT=FALSE. If no value is specified, the default of `/data` is used.

Here is an example:

```bash
SAS_DA_CONTENT_ROOT=/accounting/data
```

### Configure Access to Java, Hadoop, and Spark

The `sas-data-agent-server-colocated-config.properties` file contains configuration properties for
Java, SAS/ACCESS Interface to Spark and SAS/ACCESS to Hadoop.

#### Configure SAS_DA_HADOOP_JAR_PATH and SAS_DA_HADOOP_CONFIG_PATH

If your deployment includes SAS/ACCESS Interface to Spark, you must make your Hadoop JARs and configuration file available on a PersistentVolume or mounted storage.
Set the options SAS_DA_HADOOP_JAR_PATH and SAS_DA_HADOOP_CONFIG_PATH to point to this location.
See the SAS/ACCESS Interface to Spark documentation at `$deploy/sas-bases/examples/data-access/README.md` (for Markdown format) or `$deploy/sas-bases/docs/configuring_sasaccess_and_data_connectors_for_sas_viya_4.htm` (for HTML format) for more details. These variables have no default values.

Here are some examples:

```bash
SAS_DA_HADOOP_CONFIG_PATH=/clients/hadoopconfig/prod
SAS_DA_HADOOP_JAR_PATH=/clients/jdbc/spark/2.6.22
```

#### SAS_DA_JAVA_HOME

Use this variable to specify an alternate JAVA_HOME for use by the co-located SAS Data Agent. This variable has no default value.

Here is an example:

```bash
SAS_DA_JAVA_HOME=/java/lib/jvm/jre
```

### Configure Miscellaneous Options

#### SAS_DA_UPDATE_TENANTS

Enter a string value that indicates how tenants should be updated when the default provider is updated.

   * `SAS_DA_UPDATE_TENANTS=PROVIDER` indicates that all tenant's versions will be checked and updated if needed whenever the main provider tenant pod is restarted. 
   * `SAS_DA_UPDATE_TENANTS=ALL` indicates that all all tenant's versions will be checked and updated if needed whenever any tenant pod is restarted.
   * `SAS_DA_UPDATE_TENANTS=NONE` indicates that tenants will not be automatically updated. Administrators will need to manually run the sas-data-agent-server-tenant CronJob in order to update the tenants after the main provider tenant is updated.

If no value is specified, the default of Provider is used.

### Revise the Base kustomization.yaml File

Add these entries to the base kustomization.yaml file (`$deploy/kustomization.yaml`) in order to include 
the modified `sas-data-agent-server-colocated-config.properties` and `sas-data-agent-server-colocated-secret.properties` files.

```yaml
configMapGenerator:
...
- name: sas-data-agent-server-colocated-config
  behavior: merge
  envs:
  - site-config/sas-data-agent-server-colocated/sas-data-agent-server-colocated-config.properties
...
secretGenerator:
...
- name: sas-data-agent-server-colocated-secrets
  behavior: merge
  envs:
  - site-config/sas-data-agent-server-colocated/sas-data-agent-server-colocated-secret.properties
```

## Using  SAS/ACCESS with a Co-located SAS Data Agent

For more information about configuring SAS/ACCESS, see the README file located at `$deploy/sas-bases/examples/data-access/README.md` (for Markdown format) or `$deploy/sas-bases/docs/configuring_sasaccess_and_data_connectors_for_sas_viya_4.htm` (for HTML format).

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