# labbuilder2
Be sure to check out the [Wiki](https://github.com/parmstro/labbuilder2/wiki)

Welcome to version2 of lab builder. We have changed the project to focus on building virtual environments instead of physical environments. In its current iteration, labbuilder2 requires a VMware environment to start off. If you want to build some physical systems in place of the IdM and Satellite nodes, that shouldn't be a problem, however, it is left as an exercise for the user.

## Lab builder goals

The goal of this project is to build a Red Hat based lab environment suitable for demostrating the a Red Hat infrastructure that implements several  Standard Operating Environments for Red Hat Enterprise Linux. There will be more documentation and presentation material made available later, but for now, please check out the [Wiki](https://github.com/parmstro/labbuilder2/wiki)

When completed Lab builder should be able to deploy all parts of the standard infrastructure on any hypervisor or cloud (or combinations thereof). The basic deployment builds a lab on VMware, sets up the the hosting VMware cluster as a compute resource in Satellite and deploys the rest of the hosts on that cluster. The build is controlled by a provisioning node (usually your laptop) that kicks off the main ansible playbook, site.yml.

The lab build includes:
  - creating and downloading a base image from Red Hat image builder on console.redhat.com
  - Red Hat Identity Management primary server with "POC" users, groups, HBAC and sudo, etc..
  - Red Hat Satellite with all the bells and whistles turned on
  - Red Hat Ansible Automation Platform controllers, an execution environment and sample workflows for managing Satellite content
  - Red Hat Ansible Automation Hub
  - 2 x Container hosts that run tang servers in containers to manage NBDE for some of the sample hostgroups
  
## Lab builder non-goals

It is not currently a goal of lab builder to create a production environment, although it could do that for small environments. The flexibility to build a completely custom production environment is not there yet.


## High Level Flow

- collect the credentials and config elements needed
- plan your deployment
- configure the appropriate variable files and inventory
- run ansible-playbook -i inventory site.yml
- make coffee, pull out your favourite game console

## Preparation

We can break lab builder flow into a number of phases. 
- create the image and load it to our target
- bootstrap our idm system and configure it
- bootstrap our satellite system and configure it
- bootstrap ansible and our other hosts

Each of these phases requires configuration. We are going to keep it simple for this overview. 1 Idm server, 1 satellite, 1 automation controller, 1 hub, and 2 container hosts (we want redundancy for our tang servers). To even get started, we need to gather some credentials. 

You need a login to access.redhat.com and need to create an offline API token. The information on this process is located [here](https://access.redhat.com/articles/3626371). If you are familiar with the process, you can jump right to your [API Token management page](https://access.redhat.com/management/api). You will also need the organization number of your red hat account and the name of an activation key that will enable your IdM server and Satellite server. Attach the proper subscriptions to the activation key to enable the deployment of the two servers. These servers will start out registered directly to the CDN, once the Satellite server is built, the IdM server will unregister from the portal and re-register to the Satellite server that you build. You can plan accordingly.

You also need access to the target VMware instance that you are going to build your lab on. For now Labbuilder uses only VMware, however, we are working diligently to add other providers and will eventually support other hypervisors and the hyperscalers. For VMware, you need a service account that allows you to upload images to storage and create virtual machines. You should be able to use any account that meets the criteria for a [Satellite compute resource](https://access.redhat.com/solutions/1339483). In short in needs the following:

  - All Privileges -> Datastore -> Allocate Space, Browse datastore, Update Virtual Machine files, Low level file operations, Update virtual machine metadata
  - All Privileges -> Network -> Assign Network 
  - All Privileges -> Resource -> Apply recommendation, Assign virtual machine to resource pool 
  - All Privileges -> Virtual Machine -> Change Configuration (All) 
  - All Privileges -> Virtual Machine -> Interaction (All) 
  - All Privileges -> Virtual Machine -> Edit Inventory (All) 
  - All Privileges -> Virtual Machine -> Provisioning (All)
  
As a general note, we are using ansible vault for storing our credentials and follow a standard pattern. For each variable file that stores encrypted variables, we also have a corresponding file _vault.yml. For example:
- builder_vars.yml
- builder_vault.yml

When we have a variable secret that we are vaulting we reference a vaulted variable using variablename_vault, like this:
- offline_token: "{{ offline_token_vault }}"

To see a list of the variables used in each of the phases see the appropriate wiki page as outlined below. 

See the Wiki page for [configuring the buildimage variables](https://github.com/parmstro/labbuilder2/wiki/Configuring-the-buildimage-variables).

