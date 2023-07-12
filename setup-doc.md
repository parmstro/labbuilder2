# Getting Start with Labbuilder

## Requirements

Before starting with the LabBuilder project, you'll need to have the following completed;
- [Red Hat Login](https://developers.redhat.com/register/?intcmp=701f2000001OMHaAAO) -- Red Hat Developer
  - Learn more about [Red Hat Developer Program](https://developers.redhat.com/about)
- [Podman](https://podman.io/docs/installation)
- [Ansible-core](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
- Recommended [VSCode](https://code.visualstudio.com/download)


## Overview...

1. Fork and Clone LabBuilder Project
2. Gather Information
   -  Red Hat Offline Token
   - Create Manifest File
   - Red Hat Organization ID
   - Automation Hub Offline Token
3. Configure Variable File
4. Configure Vault File
5. Setup environment
6. Build Image -- Phase 1

## Gather Information

**Red Hat Offline token**

Visit [API Management](https://access.redhat.com/management/api) and click **Generate Token**. This will create a token to authenticate against the Red Hat API, and will expire after 30 days of activity. Be sure to copy into a secure location, because if you re-load the page you'll lose the token. You will use this within the `rhisbuilder_vault.yml` file for `offline_token`.

The instructions for how to generate and use an offline token can be found [here](https://access.redhat.com/articles/3626371).

**Manifest for AAP**

A manifest is required for the Ansible Automation Controller that will be created. This is basically a file that contains information subcriptions as a `.zip` file. Given we are using infastructure as code to create the environment, this file needs to be present before the Ansible Controller is provisioned.

To create a manifest, you must be an **Organization Administrator** within your Red Hat Organization/Account, and you must currently own a valid Ansible Automation Platform subscription.

Creating a new subscription allocation allows you to set aside subscriptions and entitlements for a system that is currently offline or air-gapped. This is necessary before you can download its manifest and upload it to Ansible Automation Platform.

**Procedure**

1. From the [Subscription Allocations](https://access.redhat.com/management/subscription_allocations/) page 
2. Click New Subscription Allocation.
3. Enter a name for the allocation so that you can find it later.
4. Select Type: Satellite 6.8 as the management application.
5. Click Create.

Reference the [documentation](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_operations_guide/assembly-aap-obtain-manifest-files) here.

**Red Hat Organization ID**

Vist https://access.redhat.com/ and login using your Red Hat loginID and password. Click in the top right hand corner on the profile icon and copy **Account Number**.

![Customer Portal](image.png)

**Activation Key**

This can be created from an existing Satellite deployment that has proper entitilments allocated or the activtion key can be created from the Red Hat [Hybrid Cloud Portal](https://console.redhat.com/). 

Create a New Activation Key 
1. From the [Activation Keys](https://console.redhat.com/settings/connector/activation-keys) page 
2. Click 'Create Activation Key'
3. Select
   - Name
   - Role
   - Service level
   -  Useage
4. Click Create

You will use the activation key name within the `rhisbuilder_vault.yml` file for `activation_key_vault`.

Reference the [documentation](https://access.redhat.com/documentation/en-us/red_hat_insights/2023/html/remote_host_configuration_and_management/activation-keys) here.

**Generate Automation Hub Offline Token**

Before you can interact with automation hub by uploading or downloading collections, you need to create an API token. The automation hub API token authenticates your ansible-galaxy client to the Red Hat automation hub server.

Procedure;

1. Navigate to [Automation Hub - Connect to Hub](https://cloud.redhat.com/ansible/automation-hub/token/)
2. Click Load Token
3. Click copy icon to copy the API token to the clipboard
4. Paste the API token into a file and store in a secure location

You will use the Automation Hub Token within the `rhisbuilder_vault.yml` file for `automation_hub_token_vault`.

**Create Automation Hub Sync List**

A synclist is a curated group of Red Hat Certified collections that is assembled by your Organization Administrator that syncs to your local Automation Hub. Use synclists to manage only the content that you want and exclude unnecessary collections.

Each synclist has its own unique repository URL you can use to designate as a remote source for content in Automation Hub and is securely accessed using an API token.

Create Unique Automation Hub URL;
- https://console.redhat.com/api/automation-hub/content/\<youraccountnumber\>-synclist/


You will use the Automation Hub Token within the `rhisbuilder_vault.yml` file for `automation_hub_url_vault`.

## Configure variable files

The defaults for the configuration file are for VMWare, and for most part unless you are deploying to AWS, the variables can remain the same.

The variables below are the ones needed to change to deploy to AWS.

- `builder_vars.yml`
  - `imagebuilder_image_type` == "aws"
  - `image_extension` == "ami"
- `init_env_vars.yml`
  - `rhisbuilder_bootstrap_target` == "aws"



## Create Vault File

We are using ansible vault for storing our credentials and follow a standard pattern. Everything needed goes into a "outside of git" vault file (rhisbuilder_vault.yml in homedir). This way the vault file doesn't wind up getting much bigger and it's useful to have the var in the same place.

```bash
# Create Vault File
touch ~/.rhisbuilder_vault.yml

# Populate rhisbuilder_vault.yml
vi ~/.rhisbuilder_vault.yml

---
aws_account_nbr_vault: 'Your AWS account number'
aws_access_key_vault: 'Your AWS access key'
aws_secret_key_vault: 'Your AWS secret key'

ec2_name_prefix: 'A unique prefix to distinguish instances in AWS. Used as the pattern name and in public DNS entries'
ec2_region: 'An AWS region that your account has access to'

pattern_dns_zone: 'A public DNS zone managed by AWS Route53'

offline_token: 'A Red Hat offline token used to download the Ansible Automation Platform bundle'

redhat_username: 'Red Hat Subscription username, used to login to registry.redhat.io'
redhat_password: 'Red Hat Subscription password, used to login to registry.redhat.io'

admin_password: 'An admin password for AAP Controller and Automation Hub'

manifest_content: "Content for a manifest file to entitle AAP Controller. See below for an example of how to point to a local file"
#manifest_content: "{{ lookup('file', '~/Downloads/manifest_AVP_20230510T202608Z.zip') | b64encode }}"

org_number_vault: "The Organization Number attached to your Red Hat Subscription for RHEL and AAP"
activation_key_vault: "The name of an Activation Key to embed in the imagebuilder image"

# Set these variables to provide your own AMI, or to re-use an AMI previously generated with this process
#skip_imagebuilder_build: 'boolean: skips imagebuilder build process'
#imagebuilder_ami: 'The ID of an AWS AMI image, preferably one that was built with this toolkit'

automation_hub_url_vault: 'The private automation hub URL associated with your Ansible Automation Platform Subscription. Used to generate ansible.cfg to retrieve Automation Hub content'
automation_hub_token_vault: 'A token associated with your AAP subscription used to retrieve Automation Hub content'


```

```bash

# Encrypt vault files
ansible-vault encrypt rhisbuilder_vault.yml
# Prompt for password x2

# Modify vault files
ansible-vault edit rhisbuilder_vault.yml

```



Reference ansible-vault [documentation](https://docs.ansible.com/archive/ansible/2.3/playbooks_vault.html) to learn more.

## Setup Environment

Within the labbuilder2 directory a script has been created to help standardize the deployment of RH-ISAM by using a container and installing all required files.

Given we are using a file encrpyted by `ansible-vault` it'll need to be decrypted by either passing `--ask-vault-pass` or storing the vault password within a file and specfiying the vault-file w/in the `ansible.cfg` file.

```bash
# Set directory
cd /Users/cshaw/Projects/labbuilder2/

# Run pre_init_env.yml to install collections
./pattern.sh ansible-playbook init_env/pre_init_env.yml
```

WARNINGS - Use the pattern.sh script to run all commands for the environment