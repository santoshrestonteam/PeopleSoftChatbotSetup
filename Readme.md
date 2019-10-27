# PeopleSoft Chatbot Setup Guide

The purpose of this guide is to provide a straight forward tutorial of how to setup PeopleSoft (also referred to as PSFT), Application Designer, Digital Assistant, Chatbot Integration Framework, and Application Services to stand up a Digital Assistant skill with PSFT. This guide assumes an introductory level of PSFT knowledge, an Oracle Cloud Account, and familiarity with the OCI console.

In the first iteration of this guide won't have screenshots. Later on I may put them in.

## Required Artifacts


1. A PeopleSoft instance with a MINIMUM of HCM PUM 31 or FSCM PUM 33 and PeopleTools 8.57.07. Earlier versions will not support the framework and you will not have the framework to simplify and secure your ODA integration. The easiest way to set this up is through OCI Marketplace.

2. An instance of Oracle Digital Assistant.

3. A Windows environment to run the Application Designer for PSFT. Either Windows Server or desktop can work. 

## Step 1: Setting Up Your PeopleSoft Instance

Follow the guide linked below to provision your PSFT instance in OCI. It will show you how to provision everything and to SSH to check on progress. NOTE: Try to avoid naming the instance with underscores. Additionally, naming subnets and your VCN will help keep the domain name short and sweet as it names it based off those two things. 

Take a note of what version # of exactly which Oracle Database it is.

https://www.oracle.com/webfolder/technetwork/tutorials/obe/cloud/compute-iaas/deploy_psft_app_marketplace_oci/deploy-psft-marketplace-oracle-cloud-infrastructure.html#section6s1

The setup of your instance will take a few minutes to complete. While you're waiting go ahead and open ports 1521 and 8000 in your new Virtual Cloud Network as those are commonly used ports with PSFT for the database and web connection.

Do this by navigating to your new compute instance and clicking the link to the associated Virtual Cloud Network. Then click security lists on the left. Open the default security list and then "Add Ingress Rule". For CIDR block put 0.0.0.0/0 and then Destination port range 1521. Add another ingress rule for port 8000.

Once completed, in your local machine navigate to /etc/hosts and add the following line:

<PUBLIC IP OF PEOPLESOFT INSTANCE> <Host Name of your instance>
Example: 150.xxx.xxx.xx psftinstance.mysubnet.myvcn.oraclevcn.com  
  
This lets you connect to your PSFT instance through the web. If you're on Windows you may need to copy this file to the desktop, edit it there, then copy it back to /etc/.


## Step 2: Getting Your Windows Environment Ready for Application Designer (You can skip this is you're on a PC)

The Application Designer is a client allowing users to develop projects, pages, application packages, etc. for their PeopleSoft instance using PeopleTools. You can connect from a Windows environment and have an IDE to start working on your projects. In our case, we'll be using it to develop Application Services that make it easier to expose PSFT components via Rest API. Then we'll use those to enable chatbot integration. Using a Windows Server is usually the fastest method for Mac/Linux users.

1. Navigate to the OCI Console. Open Compute > Instances and then "Create Instance". DO NOT PUT SPECIAL CHARACTERS IN THE NAME, JUST LETTERS. This will ensure no issues with installing Oracle DB later on. Choose Windows Server 2016 as the Image Source. Choose any AD you like, virtual machine for instance type, and then instance shape VM.Standardv2.1 or above. Choose your existing Virtual Cloud Network information. Tick the checkmark on for "Assign a public IP address", you'll need it to access your Windows Server instance.

Ensure "Custom Boot Volume Size" is a sizable amount, I did 150GB.

2. Create an SSH key pair like you did for Step 1 and then upload the .pub file.

3. Provision the instance and wait for it to come online. Once that's done follow this guide for connecting to it via Remote Desktop: https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/testingconnectionWindows.htm

## Step 3: Setting Up Application Designer

Now that you're in the Windows Server environment you can begin the process of getting Application Designer up and running. It will connect using an installation of Oracle Database to your PSFT instance so that you can work on customizing PSFT. If you're on a PC some of this info won't be required.

This video guide does a fairly good job of explaining the steps, but I'll go into a bit more detail: https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/testingconnectionWindows.htm

1. (Windows Server Only) On your Windows Server instance open the Internet Explorer application. You'll probably be restricted on what web pages you can view. To take away this limitation open Server Manager > Local Server > PROPERTIES and turn off IE Enhanced security configuration.

3. Get the SSH private key from your local machine (if you aren't on PC) and put that onto your Windows Server

4. Convert your SSH key to PPK format. Follow this guide to do so using PuttyGen: https://www.nextofwindows.com/how-to-convert-rsa-private-key-to-ppk-allow-putty-ssh-without-password

5. Download WinSCP from https://winscp.net/eng/index.php and install. After installation open the program. For the host name place the PSFT public IP, for username put "opc" without quotations, and leave password blank. Click Advanced > Authentication > and then for the Private Key file choose your private key.

6. You'll now be able to view your files in the PSFT instance. Navigate to the tools client at /opt/oracle/psft/pt/tools_client and then download the files/folders in this directory to your Windows environment into any directory you like. I used the Desktop.

7. You'll now edit the tnsnames.ora file to reflect your PSFT database information. Go to your PSFT site and open the Navbar icon on the top right and hit Navigator on the bar. Navigate to PeopleTools > LifeCycle Tools > Update Manager > About PeopleSoft Image. Keep this page open we'll need it in a bit.

8. It'll take a few seconds to load, but this will give you the database name to put into tnsnames.ora. Your file will be structured like this:

<DB_NAME> =
   (DESCRIPTION =
       (ADDRESS_LIST =
           (ADDRESS = (PROTOCOL = TCP)(HOST = <HOST_INFO>)(PORT = 1521))
       )
       (CONNECT_DATA =
           (SERVER = DEDICATED)
           (SERVICE_NAME = <DB_NAME>)
       )
    )
    
9. Change <DB_NAME> to the db name shown from the menu in step 7. Make sure to do it both at the top of the file and in SERVICE_NAME. Change the HOST value to the public IP of your PSFT instance. Save that.

10. You're almost ready to setup Application Designer! You just need to install the Oracle Database on your Windows environment.

## Step 4: Install Oracle Database

In order to work with Application Designer we need Oracle Database of the same version on our Windows Environment and configured to be connected to PSFT. 

1. Download the correct version that you previously noted from: https://www.oracle.com/database/technologies/oracle-database-software-downloads.html

2. Follow the steps for installing the database. You'll need to create a new user in the Windows environment WITHOUT admin privileges to install the db. Additionally, if you named your instance with special characters the installation won't work. 

3. Wait for the install to finish, once it does you'll be able to navigate to ORACLE_HOME\network\admin (For me it was C:\app\oracle\product\12.1.0\dbhome_2\NETWORK\ADMIN, if for some reason your directory differs you can perform a search in File Explorer for tnsnames.ora) and edit the tnsnames.ora file there and add in the entry for the PSFT database connection you placed in the other tnsnames.ora file you have in the Application Designer folder. Do not delete the existing information in the file.

## Step 5: Run the Installer for Application Designer

Now you're going to actually setup the Application Designer client so you can start working on your projects!

1. Open Command Prompt as Administrator and navigate on your Windows environment to the location you downloaded the tools client files and folders. 

2. Run the command SetupPTClient –t –l.

3. The installation for Application Designer will ask a series of questions. Here's what you should put:

	a. Deploy PeopleTools client? Y
	b. Database Platform: 1 (Oracle)
	c. Pshome: use the default
	d. Config peopletools client? Y
	e. Database name – use the name of the PSFT database in Step 3 Part 7
	f. Server name – example: instance123.sub123.sample.oraclevcn.com
	g. userId – VP1 or PS (confirm this by logging into your PSFT instance through the web)
	h. connect id: people
	i. connect password: the password you specified for connect_pwd in the user script when provisioning your PSFT instance
	j. PeopleTools Client deployment: 3
	k. Choose “n” for the next 4 questions to avoid any installation complications. 

