# IBM Netcool Operations Insight

## Introduction

IBM® Netcool® Operations Insight enables you to monitor the health and performance of IT and network infrastructure across local, cloud and hybrid environments. It also incorporates strong event management capabilities, and leverages real-time alarm and alert analytics, combined with broader historic data analytics, to deliver actionable insight into the performance of services and their associated dynamic network and IT infrastructures.

## Contents

- Chart Details
- Prerequisites
- Resources Required
- Installing the chart
- Storage
- Configuration
- Limitations
- Documentation

## Chart Details

The ibm-netcool Helm chart and its dependent sub-charts provide the capability to deploy the following Operations Management applications from the IBM Netcool Operations Insight solution:

- Netcool/OMNIbus Core
- Netcool/OMNIbus WebGUI
- Netcool/Impact
- IBM Operations Analytics Log Analysis
- DB2 Enterprise Server Edition database

For more information on how these applications work together to provide Netcool Operations Insight functionality, see
[Netcool Operations Insight documentation: Operations Management data flow](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/concept/soc_int_dataflows_noi.html).

## Prerequisites

The following prerequisites are required for a successful installation.

- A Kubernetes cluster. For more information on the cluster, see  [Netcool Operations Insight documentation: Operations Management deployment on IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/concept/soc_int_noicontphysicaldeplexample.html).
- PersistentVolume support on the cluster.
- Kubernetes command line interface (`kubectl`) installed and able to communicate with the cluster. For more information, including the required version of `kubectl`, see [Accessing your cluster using kubectl](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/manage_cluster/cfc_cli.html).
- Helm client. For more information, including the required version of the Helm command line interface, see [Setting up the Helm CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/app_center/create_helm_cli.html).
- Helm charts and images loaded into your catalog.
- LDAP service for authentication.
- _Optional_: Custom passwords set for Netcool Operations Insight applications. For more information, see [Netcool Operations Insight documentation: Overwriting default passwords](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_overwriting-passwords-icp.html).

For more information on prerequisites, see  [Netcool Operations Insight Documentation: Preparing for installation on IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/soc_int_preparing-icp-installation.html).


## Resources Required

The ibm-netcool Helm chart and its dependent sub-charts have the following minimum resource requirements.

**Note**: This minimum setup will not provide production-like
performance; however, it will be sufficient for demonstration purposes.

- 4 virtual machines in your Kubernetes cluster, assigned as follows:
    - 1 master/management/proxy/boot node
    - 3 worker nodes
- Each worker node must meet the following resource requirements:
    - 8 CPUs
    - 32 GB memory


## Installing the chart

You can install the ibm-netcool chart from the user interface or from the command line. Whenever you install the chart a new release is created. The chart can be installed multiple times into the same cluster. Each release can be independently managed and upgraded.

This section contains the following subsections:

- Installing from the GUI
- Installing from the command line
- Uninstalling the chart

### Installing from the GUI

1. In the **IBM Cloud Private** banner at the top of the page, click **Catalog** on the right hand side of the banner.

2. Enter netcool in the **Filter** field.

3. Click the **ibm-netcool-prod** Helm chart.

4. To configure the installation, click **Configure** and complete the following fields:

  - **Configuration** section

      - **Release name**: Name your release. Make sure that the name does not start with a number or a capital letter.

      - **Target namespace**: Install into the `default` namespace, or into a prespecified custom namespace.

      - Click **I have read and agreed to the license agreements**.

  - **Global** section

    - **Master node FQDN (Fully Qualified Domain Name) - Do not use IP address**: Specify the FQDN of the master node on your network. This value will be used to construct ingress URLs to access NOI services. This field corresponds to the `global.cluster.fqdn` parameter in the ibm.netcool `values.yaml` file (or in the `custom-values.yaml` file, if you copied `values.yaml` to a custom file.

    - **Enable sub-chart resource requests**: Choose whether to enable the sub-chart resource requests, or disable the requests so that there is no check on the resources required for the release. For example, to specify a small three worker node system, you must disable the sub-chart resource requests by setting this field to `false`. This field corresponds to the `global.resource.requests.enable` parameter in the ibm.netcool `values.yaml` file.

    - **Enable anti-affinity (Advanced)**: Enable this setting to prevent primary and backup server pods from being installed on the same worker node.

    - **Enable data persistence (Recommended)**: Enable this setting to ensure that data continues to be available if the pod needs to restart. If data persistence is disabled, data will be lost between pod restarts.

    - **Use dynamic provisioning (Recommended)**: Enable this setting to ensure that storage volumes are created automatically in the cluster as and when required.

    - **LDAP mode**: Choose whether to install the built-in LDAP server (openLDAP server) that comes with Netcool Operations Insight, or to install a proxy LDAP server and connect to your organization's LDAP server. This field corresponds to the `global.ldapservice.mode` parameter in the ibm.netcool `values.yaml` file.
      - `standalone`: install the built-in LDAP server
      - `proxy`: install the proxy to connect to your organization's LDAP server. **Note**: If you select this option, then you must set custom values for all of the other LDAP parameters on this screen to match your organization's LDAP server.

    - **LDAP Server URL**: If you set **LDAP mode** to proxy, then you must configure the URL of your organization's LDAP server. Use the format `ldap://<IP address or hostname>:port`. This field corresponds to the `global.ldapservice.internal.url` parameter in the ibm.netcool `values.yaml` file.

    - **LDAP Server port**: If you set LDAP mode to proxy, then you must configure the port of your organization's LDAP server.  This field corresponds to the `global.ldapservice.internal.ldapPort` parameter in the ibm.netcool `values.yaml` file.

    - **LDAP Server SSL port**: If you set LDAP mode to proxy, then you must configure the SSL port of your organization's LDAP server.  This field corresponds to the `global.ldapservice.internal.ldapSSLPort` parameter in the ibm.netcool `values.yaml` file.

    - **LDAP Directory Information Tree top entry**: If you set LDAP mode to proxy, then you must configure the top entry in the LDAP Directory Information Tree (DIT). Use the standard domain settings as configured in your organization.  This field corresponds to the `global.ldapservice.internal.suffix` parameter in the ibm.netcool `values.yaml` file.

    - **LDAP base entry**: If you set LDAP mode to proxy, then you must configure the LDAP base entry by specifying the base distinguished name (base DN).  This field corresponds to the `global.ldapservice.internal.baseDN` parameter in the ibm.netcool `values.yaml` file.

    - **LDAP bind userid**: If you set LDAP mode to proxy, then you must configure the LDAP bind user identity by specifying the bind distinguished name (bind DN).  This field corresponds to the `global.ldapservice.internal.bindDN` parameter in the ibm.netcool `values.yaml` file.

      You must also create a Kubernetes secret containing password information for your organization's LDAP server, as described in [Netcool Operations Insight Documentation: Overwriting the LDAP password](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_overwriting-ldap-password.html).

  - **WebGUI** section

    - **ASM release name**: If you plan to install the optional Service Management extension, then specify the release name that you plan to use to deploy the Agile Service Manager solution extension. Make sure that the name does not start with a number or a capital letter. When you install Agile Service Manager on IBM Cloud Private you must make sure to use release name that you specified in this field.

  - **RBAC** section

    RBAC stands for role-based access control.
    - **Create required RBAC RoleBindings. (Advanced)**: Ensure that this parameter is checked.
    This parameter ensures that the Kubernetes service account in the namespace being deployed to has the minimum required authorization to perform the installation, as described in [Netcool Operations Insight Documentation: Installing Operations Management on IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_installing-opsmg-icpt.html).


5. Do not change the values in any of the other fields.

6. Click **Install**. Monitor the progress of the installation, as described in [Netcool Operations Insight Documentation: Installing Operations Management on IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_installing-opsmg-icpt.html).

3. Review the post-installation tasks, as described in [Netcool Operations Insight Documentation: Post-installation tasks](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_post-install-tasks-icp.html).

### Installing from the command line

The helm install command is a very powerful command with many capabilities. To learn more about it, refer to the [Using Helm Guide](https://docs.helm.sh/helm/#helm-install).

1. Configure the installation by editing the `values.yaml` file (or in the `custom-values.yaml` file, if you copied `values.yaml` to a custom file.

2. Run the following command to install  Netcool Operations Insight.
```
helm install ibm-netcool -f values.yaml --tls <release_name>
```
Where:

 - `values.yaml` is the name of the configuration file containing the configuration parameters. The content of this file is described in the **Configuration** section.
 - `<release_name>` is an optional name for this release. If you do not specify a name, the system will assign a name.

3. Review the post-installation tasks, as described in [Netcool Operations Insight Documentation: Post-installation tasks](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_post-install-tasks-icp.html).

### Uninstalling the chart

Uninstall the chart by performing the following steps:

1. Run the following Helm command to determine which releases of Operations Management on IBM Cloud Private are installed.
```
helm ls --tls
```
1. Delete each release identified in the previous step by running the following Helm command against each release name in turn.
```
helm delete --purge --tls <release_name>
```
Where `<release_name>`  is the name of one of the releases identified in the previous step.

1. Repeat the previous step until all of the relevant releases are deleted.



## Storage

- Gluster-FS shared filesystem must be deployed on 3 worker nodes.
- 80 GB minimum storage on each of the 3 worker nodes.
- Gluster-FS client must be deployed these 3 worker nodes, and any other nodes in the cluster.

For more information on storage requirements and the steps required to set up your storage, see [Netcool Operations Insight Documentation: Requirements for an installation on IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/reference/soc_int_reqsforcloudinstallation.html)).

## Configuration

The following tables lists the configuration parameters of the chart and the values to which they must be set. These parameters are defined in the `values.yaml` file (or in the `custom-values.yaml` file, if you copied `values.yaml` to a custom file).

### Required parameters
You must set values for these parameters.

| Parameter name                           | Description                                        | Default   |
|--------------------------------|----------------------------------------------------|-----------|
|`global.cluster.fqdn` | **Master node hostname or IP address** Hostname or IP address of the master node in your cluster               | No default value      |
|`global.resource.requests.enable`                     | **Enable sub-chart resource requests** Flag indicating whether to enable the sub-chart resource requests, or disable the requests so that there is no check on the resources required for the release. For example, to specify a small three worker node system, you must disable the sub-chart resource requests by setting this field to `false`.                | No default value         |
|`global.ldapservice.mode`         | **LDAP mode** Flag indicating whether to install the built-in standalone LDAP server (openLDAP server) that comes with Netcool Operations Insight, or to install a proxy LDAP server and connect to your organization's LDAP server.                      | standalone|
|`global.ldapservice.internal.url`         | **LDAP Server URL** If you set `global.ldapservice.mode` to proxy, then you must configure the URL of your organization's LDAP server. Use the format `ldap://<IP address or hostname>:port`.                      | No default value|
|`global.ldapservice.internal.ldapPort`         | **LDAP Server port** If you set`global.ldapservice.mode` to proxy, then you must configure the port of your organization's LDAP server.                      | 389|
|`global.ldapservice.internal.ldapSSLPort`         | **LDAP Server SSL port** If you set `global.ldapservice.mode` to proxy, then you must configure the SSL port of your organization's LDAP server.                      | 636|
|`global.ldapservice.internal.suffix`         | **LDAP Directory Information Tree top entry** If you set `global.ldapservice.mode` to proxy, then you must configure the top entry in the LDAP Directory Information Tree (DIT). Use the standard domain settings as configured in your organization.                      | dc=mycluster,dc=icp|
|`global.ldapservice.internal.baseDN`         | **LDAP base entry** If you set `global.ldapservice.mode` to proxy, then you must configure the LDAP base entry by specifying the base distinguished name (base DN).                      | dc=mycluster,dc=icp|
|`global.ldapservice.internal.bindDN`         | **LDAP bind userid** If you set `global.ldapservice.mode` to proxy, then you must configure the LDAP bind user identity by specifying the bind distinguished name (bind DN).  You must also create a Kubernetes secret containing password information for your organization's LDAP server, as described in the [Netcool Operations Insight documentation: Overwriting the LDAP password](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/integration/task/int_overwriting-ldap-password.html))                     | cn=admin,dc=mycluster,dc=icp|
|`rbac.create`         | **Create necessary RBAC authorization** Setting `rbac.create` to true ensures that the service account in the namespace being deployed to has the minimum required authorization to install this product. This is achieved by creating two rolebindings in the namespace being deployed to. One binding ensures that the pods are allowed read/patch some of the kubernetes services created. The other binding ensures the privileged pod security policy can be used by the db2 pod. | true|


### Other parameters
Do not change the values of any of the other parameters.



## Limitations

The following limitations are associated with DB2 Enterprise Server Edition database installed by the ibm-netcool Helm chart.

### DB2 Namespace limitation

There is a namespace limitation associated with DB2. If you deploy into the `default` namespace then you will not encounter this issue. However, if you choose to deploy into another namespace, the namespace user must contain privileged PodSecurityPolicies. For more information, see [Db2 Integration into IBM Cloud Private: Namespace limitation](https://developer.ibm.com/recipes/tutorials/db2-integration-into-ibm-cloud-private/#r_step8).

### DB2 Stateful set limitation
DB2 stateful sets cannot be scaled. For more information, see [Db2 Integration into IBM Cloud Private](https://developer.ibm.com/recipes/tutorials/db2-integration-into-ibm-cloud-private/#r_step8).

## Documentation

Full documentation on deploying the ibm-netcool-prod chart can be found in the [Netcool Operations Insight documentation](https://www.ibm.com/support/knowledgecenter/SSTPTP_1.5.0/com.ibm.netcool_ops.doc/soc/collaterals/soc_netops_kc_welcome.html).
