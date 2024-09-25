---
category: SAS Event Stream Processing
tocprty: 3
---

# Adding Global Configuration Settings for SAS Event Stream Processing Projects

## Overview

Use the `$deploy/sas-bases/examples/sas-esp-operator/espconfig/espconfig-properties.yaml`
and `espconfig-env-variables.yaml` files to set the default configuration for the ESP Operator
and all ESP servers that you start within a Kubernetes environment.

Each global property and environment variable that is described in these examples
represents optional settings that enable you to change the default configuration.
If no configuration changes are required, do not add these example transformers
to the `kustomization.yaml` file.

## Prerequisites

By default, each global property setting or environment variable is commented
out. Start by determining which of the `espProperties` or environment variables
you intend to set.
The following list describes the configuration properties that can be added and
provides information about how to set them.

* `espconfig-properties.yaml`

  * Determine the `server.disableTrace` value.
    * Set to `"true"` in order to avoid log injection.

  * Determine the `server.mas-threads` value.
    * Set the number of SAS Micro Analytic Service threads that should be used across all CPUs.
    * When set to `"0"`, creates one thread per CPU.

  * Determine the `server.store-location` value.
    * Specify a full path to the location of SAS Micro Analytic Service stores.
    * This location must be available through a persistent volume.

  * Determine the `server.loglevel` value.
    * Specify a list of comma-separated key-value pairs to set the initial logging level.
    * You can set the level for individual logging contexts.
    * For example, specify `"df.esp=trace, df.esp.auth=info"`.
    * For more information, see [Logging](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=default&docsetId=espts&docsetTarget=n07870q147jx1gn1coobju0073gd.htm&locale=en).

  * Determine the `server.trace` value.
    * Specify the trace format: XML, JSON, or CSV.

  * Determine the `server.badevents` value.
    * A bad event is one that cannot be processed because of a computational failure.
    * Specify the path to the file to which bad events are written.
    * This location must be available through a persistent volume.

  * Determine the `server.plugins` value.
    * Specify any user-defined and dynamically loaded plug-ins (functions) to use.

  * Determine the `server.pluginsdir` value.
    * Specify a directory to search for user-defined and dynamically loaded plug-ins.
    * This location must be available in a persistent volume.

  * Determine the limits that are assigned to all ESP server pods and horizontal pod autoscaling.
    * In the `- op: replace` patch sections for `maxReplicas`, `maxMemory`, and
    `maxCpu` sections, you can specify the maximum number of replicas as well
    as the maximum memory and number of CPUs to allocate to the replica pods.
    * For more information about deployments, refer to the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

* `espconfig-env-variables.yaml`

  * Three environment variables are specified in the example:

    * `DFESP_QKB` - Specify the share folder under the SAS Data Quality installation.
    * `DFESP_QKB_LIC` - Specify the full file path name of the SAS Data Quality license.
    * `LD_LIBRARY_PATH` - Specify paths to append to `LD_LIBRARY_PATH`.

  * This transformer enables you to leverage additional content that
  is located in a mounted path, such as `/mnt/path/to/file`.

  * You can add any other environment variable that has not yet been
  set by using this example transformer file.

## Installation

1. Copy the files in `$deploy/sas-bases/examples/sas-esp-operator/espconfig` directory to the `$deploy/site-config/sas-esp-operator/espconfig` directory.
    Create the destination directory if it does not exist.

**NOTE:** In previous releases of SAS Event Stream Processing, this directory was named ESPConfig.

2. The file in the `$deploy/site-config/sas-esp-operator/espconfig` directory specifies the parameters of the espProperties.

    For each SAS Event Stream Processing configuration property that you intend to use, uncomment the three lines that are associated with the property: `#- op:`, `#path:`, and `#value:`.
    Then replace `{{ VARIABLE-NAME }}` with the desired property value.

    Here are some examples:

    ```yaml
    ...
     - op: add
       path: /spec/espProperties/server.disableTrace
       value: "true"
    ...
     - op: add
       path: /spec/espProperties/server.loglevel
       value: esp=trace
    ...
     - op: replace
       path: /spec/limits/maxReplicas
       value: "2"
    ...
    ```

3. Add `site-config/sas-esp-operator/espconfig/espconfig-properties.yaml`
and/or `site-config/sas-esp-operator/espconfig/espconfig-env-variables.yaml` to the transformers block of the base `kustomization.yaml` file.

    Here is an example:

    ```yaml
    ...
    transformers:
    ...
    - site-config/sas-esp-operator/espconfig/espconfig-properties.yaml
    - site-config/sas-esp-operator/espconfig/espconfig-env-variables.yaml
    ...
    ```

After the base `kustomization.yaml` file is modified, deploy the software
using the commands that are described in [SAS Viya Platform: Deployment Guide](http://documentation.sas.com/?cdcId=itopscdc&cdcVersion=default&docsetId=dplyml0phy0dkr&docsetTarget=titlepage.htm).
