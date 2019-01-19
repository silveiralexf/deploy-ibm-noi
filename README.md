# Deploy IBM Netcool Operations Insight

## Overview

On this article I'll cover how to install IBM Netcool Operations Insight (NOI) on IBM Cloud Private (ICP),  and also the steps required to to install IBM Cloud Private  (ICP) on SoftLayer VM's using Ansible.
There's already extensive documentation on both, however since tools are in constant change, I though a summary the experience and problems I had during deployment could hopefully help others.
The following will be covered by this article:

- What is ICP / NOI?
- Installing ICP CE on SoftLayer VM's with Ansible
- Configuring IBM GlusterFS chart on ICP
- Installing NOI

### What is IBM Cloud Private?

As the first Google result will tell:
>"IBM Cloud Private is an application platform for developing and managing on-premises, containerized applications. It is an integrated environment for managing containers that includes the container orchestrator Kubernetes, a private image repository, a management console, and monitoring frameworks."
But what exactly this means? In simple terms, it's a way to offer the same type of service you would have on platforms such as IBM Cloud, but privately.  By that I mean, you'll be able to setup an environment for deploying applications from a catalog, and to manage security and other stuff directly through a nice user interface or a set of auxiliary tools designed to help you out.

More information can be found at the following:

- IBM Cloud Private Overview
- IBM Cloud Private Deployment GitHub Repository

What is Netcool Operations Insight (NOI)?
As defined on IBM's website:
>IBM® Netcool® Operations Insight powered with AI and Machine learning capabilities helps reduce event noise, automatically groups events related to the same problem and provides relevant context for faster resolution, allowing you to work smarter, not harder. It provides a consolidated view across your local, cloud and hybrid environments and delivers actionable insight into the performance of services and their associated dynamic network and IT infrastructures. You can now modernize and simplify your IT Operations with greater insight into highly dynamic environments, and option for containerized deployment on IBM Cloud Private.

More information can be found at the following:

- Netcool Operations Insight Demo

### Installing ICP on SoftLayer VM's with Ansible

The idea was to deploy the tools as fast as we could to proceed testing the tools, so we choose to use SoftLayer VM's and perform the install using the Ansible Playbooks provided by IBM itself.
The deployment instructions provided by IBM are pretty straight-forward,  even if you don't have much experience using Ansible. As mentioned, I won't reproduce the exact steps that are on the document guide, but instead try making a few remarks on points that might be more prone to errors (at least for me).
Differently from what is suggested on the deployment guide, we choose creating 3 worker nodes instead of two as it was suggested by the guide, specially because the idea was to use Heketi Chart for GlusterFS storage - which requires a minimum of 3 worker nodes. Since we didn't want to have any applications on the Master node to avoid possible performance issues. This is the VM setup used in our tests:

#### Master Node

Here is the definitions for my master Node to serve as reference/example creating :

```yaml
- name: Include cluster vars
       include_vars:
         file: ../cluster/config.yaml
     - name: create master
       sl_vm:
         hostname: "{{ item }}"
         domain: ibm.local
         datacenter: "{{ sl_datacenter|default('ams01') }}"
         tags: "{{ item }},icp,master"
         hourly: False
         private: False
         dedicated: False
         local_disk: False
         cpus: 8
         memory: 32768
         disks: ['100','400']
         os_code: UBUNTU_LATEST
         private_vlan: "{{ sl_vlan|default(omit) }}"
         wait: no
         ssh_keys: "{{ sl_ssh_key|default(omit) }}"
       with_items: " {{ sl_icp_masters }}"
```

First issue when trying to create the VM's was related to the size of the disk. Originally, the idea was to have one big disk and slice it on multiple file-systems, which we soon discovered would not be possible with SoftLayer without using a custom image for the deployment.
Since VM's would be created from scratch, we soon discovered that the disk sizes needed to be exactly the same as defined on the catalog. What this means in practical terms is, for the Primary disk you can either have 25 GB or 100 GB, and from the secondary disk forward you can go from 25 GB to 2 TB, however the values should match the available sizes, meaning if you try creating a VM with 275 GB, your provisioning will fail (like mine did).
Instead, you can check the documentation below for a table offering the default sizes available:
IBM Block Storage: Provisioning Considerations

#### Worker Nodes

```yaml
- name: create workers
       sl_vm:
         hostname: "{{ item }}"
         domain: ibm.local
         datacenter: "{{ sl_datacenter|default('ams01') }}"
         tags: "{{ item }},icp,worker"
         hourly: False
         private: False
         dedicated: False
         local_disk: False
         cpus: 8
         memory: 32768
         disks: ['100','200']
         os_code: UBUNTU_LATEST
         private_vlan: "{{ sl_vlan|default(omit) }}"
         wait: no
         ssh_keys: "{{ sl_ssh_key|default(omit) }}"
       with_items: " {{ sl_icp_workers }}"
```

As mentioned on previous section, we choose to create 2 disks, one for the worker nodes itself with 100GB, and a second disk with 200GB that would be used exclusively for GlusterFS, as shown below:

#### Proxy

```yaml
- name: create proxies
       sl_vm:
         hostname: "{{ item }}"
         domain: ibm.local
         datacenter: "{{ sl_datacenter|default('ams01') }}"
         tags: "{{ item }},icp,proxy"
         hourly: False
         private: False
         dedicated: False
         local_disk: False
         cpus: 4
         memory: 16384
         disks: ['100','200']
         os_code: UBUNTU_LATEST
         private_vlan: "{{ sl_vlan|default(omit) }}"
         wait: no
         ssh_keys: "{{ sl_ssh_key|default(omit) }}"
       with_items: " {{ sl_icp_proxies }}"
```

Since we were not sure if we would need additional disk space, we provisioned a secondary disk on the proxy too, "just in case", which proved to be useless since GlusterFS worked as expected with the 3 nodes we planned.
Observations on ICP Deployment Steps…

The first thing you need to do is to clone this the repository set up to use SoftLayer CLI and Ansible, by executing the following:

```bash
$ git clone https://github.com/IBM/deploy-ibm-cloud-private.git
$ cd deploy-ibm-cloud-private
$ sudo pip install -r requirements.txt
```
The deployment repository will provide you basically with 2 Ansible playbooks:

- create_sl_vms.yml: For provisioning the VM's on SoftLayer according to the specifications defined on clusters/config.yaml
- prepare_sl_vms.yml: Prepare and install required packages for ICP

A boot or bootstrap node is used for running installation, configuration, node scaling, and cluster updates. Only one boot node is required for any cluster. A single node can be used as both master and boot, which was the case for the Ansible playbook provided by IBM.
Basically the install is done through a bootstrap image provided by IBM at the following Docker HUB link:

- https://hub.docker.com/r/ibmcom/icp-inception/

The Ansible playbook basically facilitate the connection and execution of the install from a single place, since I had direct access from my workstation to the servers I was able to start the install directly from my workstation, which is more convenient than copying files and installing stuff directly on the server. 
But its important to notice that you'll probably need to use GNU Screen or tmux to avoid loosing your session due time-out, since the whole install process might take more than an hour to finish.


"PENDING TO ADD A LOT OF STUFF"


Other important change was that the ICP version that is configured on the DockerFile is 2.1.0, we choose to test the latest version 3.1.1, in order to do the same, just have to change the image version on the DockerFile as shown below:

```yaml
FROM ibmcom/icp-inception:3.1.1 
RUN apk add --update python python-dev py-pip &&\
    pip install SoftLayer
```