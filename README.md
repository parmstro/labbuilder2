# lab_builder 2022
Welcome to lab_builer the 2022 edition

The goal of the code in this repo remains unchanged - Build a Red Hat software lab from scratch. There have been many changes to the Red Hat Portfolio and we are adjusting to meet these changes.

We are reorganizing this year to make it simpler to build a custom lab, but also be able to take advantage of a more prescribed configuration. The core of the lab is still IdM, Satellite and Ansible Automation Platform. We can think of these components as the infrastructure backplane of the lab. They provide core services (DNS, Identity, Content, etc..) as well as build services and automation. This will also allow us to more cleanly uncouple configurations to adapt to future changes.

You will find folders for each component. In each folder will be the ansible code to build each component. The goal will be to be able to combine these in a desired order to build the lab configuration of your choice. 

Details on prerquisites, system requirements, etc.. are provided in individual folders.

The chicken and egg problem still remains - creating the core, build and automation services that are necessary to build the lab. We will still have to bootstrap ourselves somehow. Some have argued to build the automation controller first, load the workflows and then use those to build all of the rest of the infrastructure. Other have argued to build the IdM server first, run the intial ansible from here and then build the automation controller cluster. The first IdM server is in essence an automation controller and then transfers this role to the cluster later in the process. I prefer the later method as it delivers core services sooner. You choose your method.

To build completely from scratch, you will still have to start with one or more bare metal servers. We are still using NUC components for most services in the lab. 


Cloud configurations are supported in this edition. 



