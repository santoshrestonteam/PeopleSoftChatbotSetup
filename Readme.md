# PeopleSoft Chatbot Setup Guide

The purpose of this guide is to provide a straight forward tutorial of how to setup PeopleSoft, Application Designer, Digital Assistant, Chatbot Integration Framework, and Application Services to stand up a Digital Assistant skill with PSFT. This guide assumes an introductory level of PSFT knowledge, an Oracle Cloud Account, and familiarity with the OCI console.

In the first iteration of this guide I won't have screenshots. Later on I may put them in.

## Required Artifacts


1. A PeopleSoft instance with a MINIMUM of HCM PUM 31 or FSCM PUM 33 and PeopleTools 8.57.07. Earlier versions will not support the framework and you will not have the framework to simplify and secure your ODA integration. The easiest way to set this up is through OCI Marketplace.

2. An instance of Oracle Digital Assistant.

3. A Windows environment to run the Application Designer for PSFT. Either Windows Server or desktop can work. 

## Step 1: Setting Up Your PeopleSoft Instance

Follow the guide linked below to provision your PSFT instance in OCI. It will show you how to provision everything and to SSH to check on progress. NOTE: Try to avoid naming the instance with underscores. Additionally, naming subnets and your VCN will help keep the domain name short and sweet as it names it based off those two things. 

https://www.oracle.com/webfolder/technetwork/tutorials/obe/cloud/compute-iaas/deploy_psft_app_marketplace_oci/deploy-psft-marketplace-oracle-cloud-infrastructure.html#section6s1

The setup of your instance will take a few minutes to complete. While you're waiting go ahead and open ports 1521 and 8000 in your new Virtual Cloud Network as those are commonly used ports with PSFT for the database and web connection.

Do this by navigating to your new compute instance and clicking the link to the associated Virtual Cloud Network. Then click security lists on the left. Open the default security list and then "Add Ingress Rule". For CIDR block put 0.0.0.0/0 and then Destination port range 1521. Add another ingress rule for port 8000.

Once completed, in your local machine navigate to /etc/hosts and add the following line:

<PUBLIC IP OF PEOPLESOFT INSTANCE> <Host Name of your instance>
Example: 150.xxx.xxx.xx psftinstance.mysubnet.myvcn.oraclevcn.com  
  
This lets you connect to your PSFT instance through the web. If you're on Windows you may need to copy this file to the desktop, edit it there, then copy it back to /etc/.


## Step 2: Getting Your Windows Environment Ready for Application Designer (You can skip this is you're on a PC)

The Application Designer is a client allowing users to develop projects, pages, application packages, etc. for their PeopleSoft instance using PeopleTools. You can connect from a Windows environment and have an IDE to start working on your projects. In our case, we'll be using it to develop Application Services that make it easier to expose PSFT components via Rest API. Then we'll use those to enable chatbot integration. Using a Windows Server is usually the fastest method for Mac/Linux users.

1. Navigate to the OCI Console. Open Compute > Instances and then "Create Instance". Choose Windows Server 2016 as the Image Source. Choose any AD you like, virtual machine for instance type, and then instance shape VM.Standardv2.1 or above. Choose your existing Virtual Cloud Network information. Tick the checkmark on for "Assign a public IP address", you'll need it to access your Windows Server instance.

Ensure "Custom Boot Volume Size" is a sizable amount, I did 150GB.

2. Create an SSH key pair like you did for Step 1 and then upload the .pub file.

3. Provision the instance and wait for it to come online. Once that's done follow this guide for connecting to it via Remote De
