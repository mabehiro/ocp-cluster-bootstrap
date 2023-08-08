## OCP Cluster Bootstrap for ACM, GitOps and ZTP
Welcome to my GitHub repository!

This repository serves as the baseline for my lab, encompassing various experiments and setups. Here, you'll find a collection of tools and scripts that form the foundation for my OpenShift testing.

## Table of contents
<!-- TOC -->
1. [Prerequisite](#prerequisite)
2. [Ansible playbooks](#ansible-playbooks)
3. [KVM Host Setup](#kvm-host-setup)
4. [Mirror Registry Setup](#mirror-registry-setup)
5. [OC-Mirror](#oc-mirror)
6. [Agent Based Install and build image](#agent-based-installer-and-build-install-image)
7. [Bootstrap Cluster and Operators](#cluster-bootstrap)
<!-- TOC -->

## Prerequisite
- The KVM host is running with [CentOS 9-Stream](https://cloud.centos.org/centos/9-stream/x86_64/images/). Also, we will use it for registry VM.
- In this lab, [mirror registry](https://docs.openshift.com/container-platform/4.13/installing/disconnected_install/installing-mirroring-creating-registry.html#mirror-registry-localhost_installing-mirroring-creating-registry) for Red Hat OpenShift is used. Download mirror-registry.tar.gz.
- Install [oc-mirror](https://docs.openshift.com/container-platform/4.13/installing/disconnected_install/installing-mirroring-disconnected.html).
- Downalod and install [the agent-based-installer](https://docs.openshift.com/container-platform/4.13/installing/installing_with_agent_based_installer/installing-with-agent-based-installer.html#installing-ocp-agent-retrieve_installing-with-agent-based-installer).

Download CentOS-Stream-GenericCloud-9-20220718.0.x86_64.qcow2 and mirror-registry.tar.gz, and locate them under the git folder after clone.

## bootsratp/cluster  
 The cluster folder serves as a central location for housing operators and configurations related to the hub cluster. As the hub cluster evolves over time, it's essential to maintain and update this folder with new operators and configurations, while also ensuring that corresponding repository updates are made, which are described as following chapters. 

~~~
cluster/
├── configuration
│   ├── advanced-cluster-management
│   ├── fakefish
│   ├── odf-operator
│   ├── openshift-local-storage
│   ├── patch-operator
│   └── topology-aware-lifecycle-manager
└── operators
    ├── advanced-cluster-management
    ├── odf-operator
    ├── openshift-local-storage
    ├── patch-operator
    └── topology-aware-lifecycle-manager
~~~

## Ansible playbooks
Four playbooks are included. \
Those four playbooks drive the bootstrap up to the point that ArgoCD gitops is ready for cluster management. 

| Playbook | Description|
| ---- | --- | 
| baremetal_setup_playbook.yaml | This playbook will configure bridge interface, install and configure libvirt and sushy on the KVM host|
| registry_vm_setup_playbook.yaml | This playbook will launch CentOS VM for the private registry with 500GB disk size on the KVM host.|
| registry_setup_playbook.yaml | This playbook will install mirror-registry into the registry VM with self-signed cert. |
| vm_setup_playbook.yaml | This playbook will launch Openshift cluster based on the image generated by Agent-based-installer. |

Currently I have modifications and manual steps between playbooks in order to make the setup generic and simplified. However, as the lab evolves, my intention is to automate these manual steps to improve reproducibility and eliminate human error. 
## KVM host setup
First step is setting up a KVM host using an Ansible playbook. This playbook will create a bridge interface, install libvirtd and configure Sushy, a tool used to emulate baremetal nodes, by leveraging the containerized Sushy tool based on [Brandon's configuration](https://cloudcult.dev/sushy-emulator-redfish-for-the-virtualization-nation/).

~~~
$ ansible-playbook -i inventry baremetal_setup_playbook.yaml
~~~
## Mirror Registry Setup
### cloud-init iso 
To configure access for a CentOS instance using cloud-init, you need to create a cloud-init ISO image containing the necessary user data and meta-data. 

~~~
$ mkisofs -output init.iso -volid cidata -joliet -rock {user-data,meta-data}
~~~
### Launch Registry VM
Proceed with setting up a private registry for our clusters. Launch a CentOS VM using the Ansible playbook. 

~~~
$ ansible-playbook -i inventry registry_vm_setup_playbook.yaml
~~~
### SSL Certificate Creation
Before configuring the mirror-registry, we need to set up the SSL certificate to secure our private registry. Follow the steps in the instructions under the ["ssl" folder](https://github.com/mabehiro/ocp-cluster-bootstrap/tree/main/ssl).
### Configure the mirror-regsitry by Ansible playbook
To install and configure the mirror-registry with SSL certificate, run the Ansible playbook as follows.

~~~
$ ansible-playbook -i inventry registry_setup_playbook.yaml
~~~
### OC Mirror 
We use the oc-mirror OpenShift CLI (oc) plugin to mirror images to the mirror registry.
To push images to the mirror-registry with oc-mirror, follow these steps:

1. In the "mirror" folder, you should find an "image-setconfig.yaml" file. Edit this file according to your needs to specify the operators and their channels that you want to mirror. Refer to the [readme](https://github.com/mabehiro/ocp-cluster-bootstrap/tree/main/mirror) in the mirror folder. 

2. Run the following command to push the specified images to your mirror-registry:

~~~
$ cd mirror
$ oc-mirror --config=./imageset-config.yaml docker://registry1.cotton.blue:8443/mirror --continue-on-error
~~~
## Agent Based Installer and build install image

Under the "installer" folder, you should find the "install-config.yaml" and "agent-config.yaml" files. Make sure these files are properly configured according to your desired OpenShift installation settings and agent configurations.

Run the following command to build the install image:

~~~
openshift-install --dir manifests agent create image
~~~

Copy the agent.x86_64.iso image to the KVM host (veterans) using rsync:

~~~
rsync --rsync-path="sudo rsync" agent.x86_64.iso mabe@veterans:/var/lib/libvirt/images/
~~~
### Extra Manifests (Optional):
If you want to include extra manifests in the install image, you can follow the steps from [my blog](https://cloudcult.dev/ocp-agent-based-install-with-extra-manifests-calico/) to add those manifests to the image.
## ACM VM setup and Install Cluster 
This playbook will create VMs with the specified configurations for 3 master nodes and 3 worker nodes. The agent ISO will be attached to each VM, initiating the installation process for the OpenShift Cluster. The cluster installation process will begin after starting the VMs. 

~~~
ansible-playbook -i inventory vm_setup_playbook.yaml -K
~~~
## Cluster bootstrap 
Great! \
Now that our OpenShift Cluster is up and running, we can bootstrap the Advanced Cluster Management (ACM) and other components using the GitOps operator. \

The provided command will apply the necessary configurations to deploy the App of Apps pattern application on top of Gitops operator:

~~~
oc apply -k cluster-bootstrap/
~~~

> If the oc apply -k cluster-bootstrap/ command appears to be stuck or takes an unusually long time to complete, apply each configuration separately using the -k option to narrow down the cofiguration for specific need and operator. 

![app of apps](/images/600b587a00014f568a7dbb2090ffae10.png)

---
#### Dry run, -K asks password for sudo
~~~
ansible-playbook -i inventory baremetal_setup_playbook.yaml --check -K
~~~
#### Jinja test
~~~
ansible all -i "localhost," -c local -m template -a "src=./templates/vm_master.xml.j2 dest=./test.txt" --extra-vars='{"masters": ["master1", "master2", "master3"]}'
~~~

