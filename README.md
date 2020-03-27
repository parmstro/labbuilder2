# lab_builder
This project builds a Red Hat based lab environment

The lab build includes:
  - Red Hat Identity Management Master and "POC" users, groups, HBAC and sudo
      The initial IdM Master is the buildmaster
  - Red Hat Satellite 6.6 with all the bells and whistles turned on
      The initial Satellite server is the foreman

  The additional platforms installed will be integrated with the above
  - Red Hat Hyperconverged Infrastructure (RHV+Gluster)
  - Red Hat Ansible Platform (Tower cluster)
  - Red Hat Openshift Container Platform
  - Red Hat Openshift Container Storage

The requirements below are for a bare metal lab. We will add alternate specs
for a cloud based lab at a later date.

If you are not interested in building a bare metal lab, this project is
probably not what you want to use. Just sayin.

# Basic Process

This process was originally designed to start on bare metal systems that the 
user has direct access to - like my lab. The project includes additional 
playbooks so that the user can set this up on remote machines where access 
is limited to network only. The process requires the two bootstrap systems 
to have access to the Red Hat CDN and to have access to the appropriate 
Red Hat subscriptions to build the lab. Please see the Requirements section 
to understand which subscriptions need to be available. 

The assumptions for this process are that you have nothing but the required 
hardware to build the lab that you are specifying and a connection to the 
internet.

Minimum Environment

1 IdM Master - buildmaster
(1x 2-core proc, 16GB RAM, 128GB SSD recommended)

1 Satellite Server - foreman
(2x 2-core proc, 32GB RAM, 1x128GB SSD, 1x 1TB SSD minimum)

3 RHV Hypervisors configured for RHHI 
(ea. 2x 2-core proc, 32GB RAM, 1x128GB SSD, 1x 1TB SSD minimum, 3x 1Gb NIC)

This process has been tested on Intel NUC Gen5 and above with good success.
Any similar or more powerful systems will work. I am using NUCs because they
are cheap, quiet and consume very little power. This is for a learning lab, 
not production.

:-)

## Preparation and Phase 1

Phase 1 creates the buildmaster and foreman systems from base RHEL7 images.
At the time of writing, Satellite 6.6 does not support RHEL8. Before Phase 1
begins, we need to prepare the systems with connections to the CDN and network
information. 

There is a single image that is used to create the buildmaster and the foreman.
The root user of the image can ssh without password to the image, therefore, to
both the buildmaster and the foreman. If you are building without the image,
there are additional steps to ensure passwordless ssh for the root user. 

Directions to build your own image and a kickstart are provided in the 
labbuilder2_image repository on my github. This will require appropriate 
RHEL subscriptions in order to register. This process was not intended to use
CentOS, Fedora, or other distributions. This process is for RHEL.

To start we run the prepare.yml ansible playbook on each host. This will ask
for the information to configure itself and for the information of the
corresponding bootstrap partner. 

The buildmaster installs an IdM Master server on itself with DNS services and 
a certificate authority, configures a set of users and groups for the realm,
adds a basic set of sudo rules, and other IdM configuration that might be
useful to demonstrate or we want to include with other systems in our lab. This
configuration can easily be extended.

The buildmaster then configures the "foreman" server for prerequisites, installs 
the satellite binaries, copies over a satellite installation template and launches
the satellite installer. After the satellite installation, the buildmaster
applies the foreman-post role to the foreman server. This takes a long time to
complete, usually several hours as the system downloads all the necessary
content from the CDN. It configures all of the content, content views,
installation media, partition templates, provisioning templates, etc.. that we
will require to deploy other systems in the lab. One of the important parts of
the configuration if foreman-discovery so that we can provision the remaining
physical hosts.

Phase 1 ends when the foreman-post role is complete and the user is asked to
power on the remaining physical systems.

The buildmaster then waits for the systems to be discovered.

## Phase 2

Phase 2 begins when sufficient hosts are discovered to start the deployment of
our hypervisor platform for the lab. For default configuration is to deploy 3
hypervisor systems as Red Hat Hyperconverged Infrastructure hosts. The default
configuration expects them to have a minimum of 2 physical disks (1 being at
least 1 TB - should be SSD for performance), 32 GB of RAM and 3 NICs to support
an ovirtmgmt/vm network, a storage network and a migration network. For small
labs you can get away with a single NIC, but it is a very limited deployment.

Phase 2 builds the RHHI platform using the foreman host. The build supports the
current standard number of RHHI nodes. After the RHV nodes are deployed from 
baremetal, they are updated, ssh configured, and the appropriate templates
are deployed to the first hypervisor to guide the gluster and hosted engine
installation and configuration. Once the hosted engine is up, further
configuration is performed to integrate the hypervisors and rhvm into the IPA
configuration and ipa authenticated ssh and IPA SSO and certs for the RHVM
host. The storage domains are then verified, the Storage and Migration networks
are configured, satellite integration is configured from the RHVM side. 

Once the RHVM configuration is complete, Phase 2 continues to configure the 
foreman server to use the RHHI platform as a compute resource. Compute profiles
are created for some defined system types that we demonstrate (CPGPC profiles,
database server profiles, etc..), host groups are created that use these
profiles and the RHV compute resource, virt-who is configured on the Satellite
and finally, a test server is deployed on the RHHI compute resoure. 

At this point we are ready to deploy any remaining systems that we wish to
deliver in our lab. This is the role of Phase 3.

## Phase 3

Phase 3 is the portion of Lab Builder that deploys some additional Red Hat
services that we would like to demonstrate and also sets up the automated care
and feeding of our lab. This becomes the job of Ansible Tower and some
additional projects that can be consumed from github. 

Phase 3 begins with the deployment of Ansible Tower as a virtual machine on the
RHHI platform completed in Phase 2. 

- deploy single ansible tower node or cluster
- configure tower for authentication with IPA
- configure tower to use Satellite as an inventory source
- configure tower to manage Satellite content view publication and promotion
- configure tower to manage Satellite content view testing and QA builds
- configure Satellite to use Tower callbacks.

Phase 3 is complete when tower has successfully updated and published all
content views on Satellite and successfully run the smoke tests for the test
servers.

We are now ready to continue to any additional phase that you wish to define.

## Phase 4

The default Phase 4 deploys OpenShift Container Platform with OpenShift
Container Storage.


## Phase 5 

The default Phase 5 deploys sample infrastructure hosts such as 
- a mail server for recieving notifications from all our infrastructure.
- a rsyslog server for managing centralized logging.
- a git server for managing projects
- deployment of several OpenShift demonstration projects
- deployment of the LabBuilder Documentation.

When we are done, LabBuilder should be able to deploy LabBuilder...


