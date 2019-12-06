# PeopleSoft Chatbot Setup Guide

The purpose of this guide is to provide a straight forward tutorial of how to setup PeopleSoft (also referred to as PSFT), Application Designer, Digital Assistant, Chatbot Integration Framework, and Application Services to stand up a Digital Assistant skill with PSFT. By the end of it you'll have a sample implementation setup for FSCM (Financials) module, but you can easily modify it to suit your HCM/ELM needs.

This guide assumes an introductory level of Linux/Windows knowledge, an Oracle Cloud Account, and basic familiarity with the OCI console.

In the first iteration of this guide won't have screenshots. Later on I may put them in.

Helpful links:

-PSFT Chatbot Architecture Documentation: https://docs.oracle.com/cd/F20644_01/hcm92pbr31/eng/hcm/ecch/concept_UnderstandingSecurityForTheChatbotIntegrationFramework.html?pli=ul_d29e210_ecch

Author: Charles Moore (charles.d.moore@oracle.com)

## Required Artifacts


1. A PeopleSoft instance with a MINIMUM of HCM PUM 31, ELM PUM 20, or FSCM PUM 33 and PeopleTools 8.57.07+. Earlier versions will not support the framework and you will not have the framework to simplify and secure your ODA integration. The easiest way to set this up is through OCI Marketplace. NOTE: Only the HCM image as of the writing of this guide has a delivered skill with existing code/implementation. 

2. An instance of Oracle Digital Assistant.

3. A Windows environment to run the Application Designer for PSFT. Either Windows Server or desktop can work. 

4. The sample PeopleCode is included within the repository for retrieving all expenses for an employee in the FSCM module.

## If you're only trying to standup the HCM Delivered Absence Assistant skip to Step 6

## Step 1: Setting Up Your PeopleSoft Instance

Follow the guide linked below to provision your PSFT instance in OCI. It will show you how to provision everything and to SSH in to check on progress. NOTE: Try to avoid naming the instance with underscores. Additionally, naming subnets and your VCN will help keep the domain name short and sweet as it names it based off those two things. 

Take a note of what version # of exactly which Oracle Database it is. ONLY THE HCM IMAGE CURRENTLY HAS A DELIVERED SKILL. Keep that in mind if you'd like to see a fully fleshed out implementation of a skill vs. this guide's sample.

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

3. Get the SSH private key from your local machine (if you aren't on PC) and put that onto your Windows environment

4. Convert your SSH key to PPK format in the Windows environment. Follow this guide to do so using PuttyGen: https://www.nextofwindows.com/how-to-convert-rsa-private-key-to-ppk-allow-putty-ssh-without-password

5. Download WinSCP from https://winscp.net/eng/index.php and install on Windows environment. After installation open the program to connect to your PSFT instance in the Windows environment. For the host name place the PSFT public IP, for username put "opc" without quotations, and leave password blank. Click Advanced > Authentication > and then for the Private Key file choose your private key.

6. You'll now be able to view your files in the PSFT instance. Navigate to the tools client at /opt/oracle/psft/pt/tools_client and then download the files/folders in this directory to your Windows environment into any directory you like. I used the Desktop.

7. You'll now edit the tnsnames.ora file to reflect your PSFT database information. Go to your PSFT site and open the Navbar icon on the top right and hit Navigator on the bar (from here on this is the main means of navigation in the PSFT site). Navigate to PeopleTools > LifeCycle Tools > Update Manager > About PeopleSoft Image. Keep this page open we'll need it in a bit.

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

	1. Deploy PeopleTools client? Y
	2. Database Platform: 1 (Oracle)
	3. Pshome: use the default
	4. Config peopletools client? Y
	5. Database name – use the name of the PSFT database in Step 3 Part 7
	6. Server name – example: instance123.sub123.sample.oraclevcn.com
	7. userId – VP1 or PS (confirm this by logging into your PSFT instance through the web)
	8. connect id: people
	9. connect password: the password you specified for connect_pwd in the user script when provisioning your PSFT instance
	10. PeopleTools Client deployment: 3
	11. Choose “n” for the next 4 questions to avoid any installation complications. 

4. The client will now be installed. Once completed it should create a new directory in the C:/PT8.57.08_Client_ORA.

5. Navigate to that directory and then to C:/PT8.57.08_Client_ORA/bin/client/winx86

6. You can access Application Designer using pside.exe. 

7. If there's an error for a timeout ensure your security lists and PSFT VM's firewalls allow traffic on port 1521 and 8000. If it's an ID/password error ensure your connect ID and password are correct.

8. You can update the connect password and other information using C:/PT8.57.08_Client_ORA/bin/client/winx86/pscfg.exe. This is a program for specifying database, password, etc. information for PSFT. If you receive errors trying to login with pside.exe ensure your tnsnames.ora files are updated correctly and the connect password is right. 

## Step 6: Setting Permissions for Your PSFT Proxy User

To ensure your chatbot is secure and safe you'll need to specify a handful of permissions for a few users. You'll have a proxy user handling interactions between Application Services and ODA, an administrator for Application Services, an admin for chatbots in PSFT, and then the end chatbot user.

1. Navigate in the navbar to PeopleTools > Security > User Profiles > User Profiles

2. Click "Add a New Value" and enter PROXY_USER

3. In the "General" tab in Symbolic ID choose SYSADM1. Enter a password 'PROXY_USER' in "New Password" and "Confirm Password" to set the login credentials. After typing it in the "Confirm Password" field click any other field such that the dot mask for the two passwords increases in size. This will indicate PSFT has acknowledged the entry. 

4. Click the "ID" tab and in the drop down for ID Type choose "None". 

5. Move to the "Roles" tab and in the table under "Role Name" for the first entry type "PTCB_USER" and then in that row click the "+" button. You should see a description of Chatbot User for the first one now. In the second row's search bar look up "EOCB Service User" and select it. Move to a different tab then back to the "Roles" tab to ensure everything's populated.

6. Click "Save"

## Step 7: Specify Your Chatbot User, Application Services and Chatbot Administrator

1. Navigate in the navbar to PeopleTools > Security > User Profiles > User Profiles

2. Search for the user you are logged in as (VP1 or PS) and select it. 

3. Navigate to the "Roles" tab and add PTCB_ADMINISTRATOR (Application Services Admin), EOCB Admin User (Chatbot Admin), and EOCB Client User (Chatbot user). 

4. Click Return to Search and then Add a New Value. Enter "PROXY_USER" or something similar

5. Go to the "Roles" tab and add PTCB_USER and EOCB Service User. Make sure to save your changes

## Step 8: Create a Permission List for Your New Bot

1. Navigate to PeopleTools > Security > Permissions and Roles > Permission Lists.

2. Create a new Permission List for your new bot. I named mine EXPAMCB01.

3. Navigate to PeopleTools > Security > Permissions and Roles > Roles

4. Search for EOCB Service User and then add the newly created EXPAMCB01 permission list

## Step 9: Set Up the Web SDK on your PSFT instance

Digital Assistant utilizes a Web Software Development Kit (SDK) to handle the heavy lifting of managing the chat widget/page in PSFT. You'll be using a Web Channel to connect with your ODA as PSFT provides the website service. Here we'll put the SDK onto PSFT and then restart our web server.

1. Use SCP to copy the ochatjs.zip file included in this repository to your PSFT instance. Remember what fully qualified path you place it in within the PSFT instance. Example: scp -i ~/priv-ssh-keyfile /local_path_to/ochatjs.zip opc@ipaddress:/path_to/location_on/PSFT

2. On your local machine open an SSH session with your PSFT instance. E.g. ssh -i ~/priv-ssh-keyfile opc@ipaddress

3. Navigate to your PORTAL.war file using "cd /home/psadm2/psft/pt/8.57/webserv/peoplesoft/applications/peoplesoft/PORTAL.war/".

4. Run mkdir external.

5. Run cd external.

6. Run mv /path/to/file/ochatjs.zip /home/psadm2/psft/pt/8.57/webserv/peoplesoft/applications/peoplesoft/PORTAL.war/external/

7. Run "ls" to check if the zip file is there. 

8. Run unzip ochatjs.zip to unzip the SDK files and run the following commands:

9. Sudo –s 

10. Su psadm2. You’ll get an error message about password but it will not make a difference

11. Cd $PS_HOME/appserv

12. ./psadmin

13. Select 3 for Web Server

14. Choose administer web server

15. There should be only one web server called peoplesoft, select it

16. Choose to shutdown this domain

17. After that finishes choose 1) for Boot this domain. Your ODA Web SDK is now setup.

## Step 10: Setting Up the Application Services

Now we get to do a little bit of coding! PSFT uses a language called PeopleCode (quite original if I say so myself) to express business logic. We're going to need to use that Application Designer client to actually set that up with our sample Expense retrieval. 

1. Open your Windows environment. Execute the file pside.exe

2. Login to PeopleTools using Connection Type - Oracle / Database Name - Find this on Step 3 Part 7 / User ID - VP1 or PS / Password - The User ID Password

3. Click File > New and select Application Package

4. Right click the app package and select "Insert App Class"

5. Provide a name for your class (GetExpenseReports)

6. Copy the code accompanying this guide

7. Notice some of the API's and services:

```
1.	%This.ServiceAPI.insertOutputRow();
	Allows you to setup a multi-row output

2.	%This.ServiceAPI.OutputRows [x].setParameter("outputparameter", &variable);
	This lets you set the output parameter on a key for the output

3.	%This.ServiceAPI.ResultState = "Pass";
	This indicates the result state

4.	%This.ServiceAPI.setOutputParameter("outputparameter", &variable);
	allows you to specify a single output parameter


```

## Step 11: Working with Application Services Wizard
Here we are actually going to setup the Wizard in the PSFT to let us use the code we added to the Application Services.

1. Navigate in PSFT web to Navigator>PeopleTools>Integration Broker>Web Services>Application Services>Application Services

2. You'll see a few predefined services for chatbots

3. Click "Create Application Service". Give an App Service ID such as EX_CHATBOT_GET_REPORTS

4. The service type will be master, enter in a service URL such as expense.GetExpenseReports

5. Give a good description of what this services will provide

6. Root package ID: Search for the app package you setup in PSFT Application Designer

7. The path will be a colon, ':', without quotes

8. Application Class ID will be GetExpenseReports

9. Service cache support: none and status: active

10. Hit next, select "Yes" for multi row output, "No" for multi row input. The parameters will be as shown for getting reports

![Parameters](https://github.com/restonappdev/PeopleSoftChatbotSetup/blob/master/Expense%20Reports%20Parameters.png?raw=true)

11. Hit next and determine your result states:

![Result States](https://github.com/restonappdev/PeopleSoftChatbotSetup/blob/master/PeoplesoftResultState.png?raw=true)

12. Set the Permission List to EXPAMCB01 and continue

13. NOTE: If an existing permission list isn't working with your application service, just make a new permission list and ensure role "EOCB Service User" has that list added to it as we did before

14. Turn "token required" on

15. Confirm everything looks good and hit Submit

## Step 12: Interacting with the REST Client
Application Services provides a simple REST service to interact with. There are a couple ways to interact with it, you can use Postman to test along with utilizing an Oracle Digital Assistant.

1. Download and setup Postman if you don't already have it. This'll let you test your application services

2. Navigate in PSFT to PeopleTools > Integration Broker > Integration Setup > Services

3. Enable the REST Service checkbox and search for PTCB_APPL_SVC

4. Choose the PTCH_APPL_SVC and in the next screen click on PTCB_APPL_SVC_GET.v1 option in Existing Operations

5. Choose req verification - basic authentication (only use SSL if your PSFT environment begins with HTTPS)

6. Click validate for both the URI's describe and describe/{service_id}

7. For both choose "generate url" and store each

8. Open Postman and create a new request

9. Set the method to GET, in the url specify the "describe" get request URL you saved previously

10. Under the "authorization" tab specify basic auth and provide the username and password (in this case PROXY_USER / PROXY_USER respectively)

11. Click Send and you'll get a JSON response describing your application services

12. You can similarly add the parameter for one of your services described by "IDForServiceURL". E.g. <URL>/expense.GetExpenseReports
	
13. This will return metadata for your specific service

## Step 13: Test your Application Service

1. Create a new request in Postman and set the method to POST

2. Use the same URL for the previous requests you've made

3. Under authorization tab provide the PROXY_USER credentials

4. In the body tab ensure "raw" is selected and in the dropdown on the right change from "Text" to "JSON"

5. Include the following JSON in the body. You can retrieve PS Token by going to your PSFT, right clicking, and selecting "Inspect". Navigate to the Application tab. On the left choose "Cookies" and select the field in the drop down. Now copy the value for PS TOKEN, ensure you get the full value. For EmplID navigate to the home page and go to My Profile. Your EmplID (Employee ID) will be listed

```
{
	"AuthDetails": {
		"SwitchUserContext": true,
		"AuthOption": "PSToken",
		"Token": "<YOUR-PS-TOKEN>”
	},
	"Language": {},
	"ServiceInputParameters": {
		"EmplId": "<YOUR-EMPLID>"
	}
}

```

6. Press the "Send" button

7. You should receive a response stating Authorized and ParseStatus as "true". If Authorized is false your PS Token is expired or the EOCB Service User role needs to be applied to the user

8. If all goes well you'll see a JSON array in "ServiceOutputParameters" of your expense reports

## Step 14: Get the template skill

1. Use SSH to connect to your PSFT instance as you did before

2. Navigate to <PS_APP_HOME>/setup/chatbot, if it's not showing up . Once you're there use scp to download this template skill. E.g. scp opc@ipaddress:<PS_APP_HOME>/setup/chatbot/where/to/put

## Step 15: Setup the ODA Skill
Now you're ready to setup the ODA skill and connect it with Application Services. Please read this carefully as there are some details that will depend on your PSFT setup.

1. Login to your OCI account and open the hamburger menu on the top left. Either choose Digital Assistant from this menu, or select "Platform Services" and select it from there. 

2. Follow this documentation on provisioning and IDCS (Ensure you have the adequate privileges to operate this service): https://docs.oracle.com/en/cloud/paas/digital-assistant/service-administration/get-started-oda.html#GUID-AAA43B23-C55A-45F2-80F6-1EFF94FAE061

3. In the ODA service click the hamburger menu on the top left > Development > Skills

4. Click on the import skill button to import the template skill. Navigate to where you downloaded the zip file on your machine. Import the zip file included here

5. Download nodejs on your local machine and install. See Nodejs's website for how to do this: https://nodejs.org/en/download/

6. Unzip the expenses bot folder in your machine. Open the directory in a terminal or command prompt window. Ensure you're in the same directory as package.json. Run command "npm install" to install all dependencies on your machine

7. Edit the file psft-lib > config > environments.js. In our case we want to update the info for our PSFT environment. If you aren't using HCM you can delete or ignore the object

8. The code structure is as follows:

```
{
    "HCM": {
        "baseURL": "HOSTNAME:PORT/PSIGW/RESTListeningConnector/HCM_JQWCFCVPSVG/PTCB_APPL_SVC.v1/describe",
        "user": "PROXY_USER",
        "password": "PROXY_USER",
        "static_describe": "../config/hcm_describe.json"
    },
    "FSCM": {
        "baseURL": "HOSTNAME/PSIGW/RESTListeningConnector/FSCM_NVASRRNKXQ/PTCB_APPL_SVC.v1/describe",
        "user": "PROXY_USER",
        "password": "PROXY_USER",
        "static_describe": "../config/fscm_describe.json"
    }
}

```

9. If you are running a PSFT instance without an HTTPS site name, use the public IP of your PSFT deployment and the port your PIA is running on (usually 8000) in baseURL as shown in the HCM example. If https then put the host name (e.g. https://psft-instance.com:8000

10. Ensure the proxy user information is accurate

11. Save the file and then open psft-lib > config > fscm_describe.json. Use the response you got in Postman from Step 12 part 11. This is what the custom component will use to create services for you (saves a lot of coding time!)

12. Save and navigate in Terminal or Command Prompt to the same directory as package.json. Run the command "npm pack". Make sure there are no errors in the logs and you'll see a .tgz file

13. Navigate in ODA to the skill you imported and then to the icon above the gear (looks like an "F"). You'll see a custom component package in there. Click "change" and upload the newly created .tgz file

## Step 16: Configure your skill

1. Navigate to the gear icon and then to the "Configuration" tab. Go down to custom parameters and put in parameters for PSFSCMbaseurl, PSFSCMpassword, and PSFSCMuserid

2. Put each as string and enter in your proxy user's info and the url you used before as shown here: https://docs.oracle.com/cd/F20644_01/hcm92pbr31/eng/hcm/ecch/task_ImportingAndSettingUpADeliveredSkill.html?pli=ul_d29e210_ecch

3. Take care to leave off the "/" at the end of the baseurl. If your connection is NOT https then use the structure http://<PSFT-PUBLIC-IP:PORT/PSIGW/RESTListeningConnector/FSCM_NVASRRNKXQ/PTCB_APPL_SVC.v1

4. In the hamburger menu navigate to Development > Channels

5. Create a new channel and specify "Web"

6. Take note of the App Id once provisioned and activate the channel

7. Route the channel to your newly created skill 

## Step 17: Setup the bot in PSFT

1. In PSFT navigate to Enterprise Components > Chatbot Configurations > Bot Definitions

2. Add a new record for your bot. For ID, you can say something like EX_CHAT_ASST

3. Once added you can specify more information, you'll need to include the App ID from your ODA channel

4. Keep the defaults as shown here: https://docs.oracle.com/cd/F20644_01/hcm92pbr31/eng/hcm/ecch/task_CreatingBotDefinitions.html?pli=ul_d29e210_ecch

5. Ensure you have a role configured named "Expense Chatbot Employee". If not create one in PeopleTools > Security > Permissions and Roles and ensure permission lists EOCB_CLIENT_USER and EXPAMCB01 are added.

6. Add that role in the Roles section of the Maintain Bot page

## Step 18: Add the widget

1. In the search bar look up "Component Mapping" to get to the component mapping page

2. Click Add to choose a new place to setup your chatbot widget

3. Choose EX_EXP_LIST_FL in the market GBL

4. Open the mapping up and then in Associated Bots choose your EX_CHAT_ASST

5. Save your changes and navigate to PeopleTools > Portal > Related Content Service > Manage Related Content Service

6. Choose event mapping and then Map Application Classes to Component Events

7. Choose Fluid Structure Content > Fluid Pages > Travel and Expenses > My Expense Reports 

8. Follow the picture below to setup the mapping. Field name: Variable / Event Name: PostBuild / Service ID: EOCB_CHATCLIENT / Processing Sequence: PostProcess

![PSFTSetup](https://docs.oracle.com/cd/F20644_01/hcm92pbr31/eng/hcm/ecch/img/i53c3884en-71f1.png)

9. Save and your widget will be available in your Expense Reports page





