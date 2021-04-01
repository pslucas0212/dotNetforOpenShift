# Run .Net code on Red Hat OpenShift Container Platform
Create a simple Hello World .Net 5 application and run it on Red Hat OpenShift (Code Ready Container example)

### Pre-req .Net 5 SDK
Download and instll .Net 5 SDK for your OS -> https://dotnet.microsoft.com/download/dotnet/5.0

## Install Red Hat CodeReady Containers
Follow the instructions Create an OpenShift cluster -> https://cloud.redhat.com/openshift/create/local to install CodeReady Containers (CRC). Chose the "local" tab and select your OS.

## Prep your CodeReady Containers environment
Set up CRC:
      
        # crc setup
        
## Start up CRC
Start up crc an indluce the pull secret you previously downloaded

        # crc start -p ~/Documents/crc/pull-secret.txt

Copy the information regarding url to the OpenShift (OCP) console, and the ids and passwords for the admin and developer

## Create a sample Hello World .Net web app
I created my sample Hello World .Net web app in directory called projects

       # dotnet new webApp -o myWebApp --no-https
       
Test your sample application.

       # cd myWebApp
       # dotnet run
       
Follow the instructions for the sample URL.
When you are finished type ctrl-c to stop server

## Optional: Modify the .Net code
Let's change the "Welcome" message to "Welcome from OpenShift Container Platform!".
Navigate the Pages directory under your project.  In the Pages directory open the file titled Index.cshtml with your favorite editor.
