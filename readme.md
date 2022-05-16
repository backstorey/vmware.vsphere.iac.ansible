# vSphere 'Desired State' System Using Ansible

> This system is in an MVP (minimum viable product) stage which delivers simple desired state of vSphere hosts and cluster config.

*...in a hurry?, jump ahead to the [getting started](#getting-started) section*
## System Overview
This vSphere IaC repo is an Ansible based system which leverages GitHub actions and the Community VMware module to configure vSphere hosts and clusters, and manage ongoing configuration drift of these objects.

## Features

Currently, the system supports config drift remediation of the following settings:

- NTP Client Configuration
- ESXi Host Advanced Settings
- vSphere DRS Config
- vSphere DRS Affinity + Anti-Affinity Groups
- vSphere HA Config
## Getting Started
To start managing hosts and clusters with this repo, you need:

- One or more YAML definition files, placed in the appropriate directories below `/vars`
- A GitHub actions workflow

A YAML hosts + cluster definition feeds into an Ansible role (vsphere.hosts.clusters).  This Ansible role handles the grunt work of communicating with vCenter and appropriately configuring hosts and clusters according to the YAML spec.

### YAML Definitions

YAML definitions are placed into the folder structure below ./vars :

- `./vars/global`
- `./vars/datacenters`
- `./vars/clusters`

#### Inheritance

The purpose of breaking these out into global / datacenters / clusters directories is to provide an inheritance mechanism.  For example, it's likley that you may have a common set of advanced parameters you wish to apply to *all hosts globally*.  Likewise, it's likely that syslog and NTP settings may be the same across multiple hosts in the *same datacenter*.

By setting these at the appropriate level, you can minimise repeated code in your YAML defs.  

Setting parameters at the cluster level will always win over parameters set at higher levels.

#### YAML Def Structure

The template file in `./vars/clusters/vsphere_iac_datacenter_cluster_template.yml` provides a good starting point for populating IaC data for a cluster you'd like to configure.  The naming of these files is critical, since this the role's `main.yml` file looks for files named after the Datacenter name and the Cluster name you're configuring.

Name these files accordingly:

`./vars/datacenters/vsphere_iac_<DATACENTER_NAME>.yml`

`./vars/datacenters/vsphere_iac_<CLUSTER_NAME>.yml`

For example, to configure a cluster called 'Cluster1' in a datacenter called 'MYDC':

`./vars/datacenters/vsphere_iac_MYDC.yml`
`./vars/datacenters/vsphere_iac_Cluster1.yml`

Case is important here.  Ensure the files match the case of the Datacenter and Cluster objects they relate to in vCenter.

# GitHub Actions

GitHub Actions is a great way to run this playbook on a schedule, executing against groups of clusters / hosts you have definitions for.  You will need one workflow per cluster, and configure the workflow's `env:` section with the appropriate `datacenter` and `cluster` environment variables.  Assuming you have definitions in the `/vars/` directory for the objects you're configuring, and you have appropriate GitHub actions secrets set for `VCENTER_USERNAME` and `VCENTER_PASSWORD`, you should be good to go.

Be sure to checkout the example actions workflow in `/example-githubactions-workflow.yml` file to help get started.  Copy this into `.github/workflows/`, and rename + edit according to your environment.

TODO:
- Add option for setting heartbeat datastores
- Add option for settings advanced cluster settings