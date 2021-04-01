# Run .Net code on Red Hat OpenShift Container Platform
Create a simple Hello World .Net 5 application and run it on Red Hat OpenShift (Code Ready Container example)

### Pre-req .Net 5 SDK
- Note: I'm using a Mac for this example.
- Download and instll .Net 5 SDK for your OS -> https://dotnet.microsoft.com/download/dotnet/5.0

## Install Red Hat CodeReady Containers
Follow the instructions Create an OpenShift cluster -> https://cloud.redhat.com/openshift/create/local to install CodeReady Containers (CRC). Chose the "local" tab and select your OS.

## Prep your CodeReady Containers environment
Set up CRC:
      
        # crc setup
        
## Start up CRC
Start up crc an indluce the pull secret you previously downloaded

        # crc start -p ~/Documents/crc/pull-secret.txt

Copy the information regarding URL to the OpenShift (OCP) console, and the ids and passwords for the admin and developer

## Create a sample Hello World .Net web app
I created my sample Hello World .Net web app in directory called projects

       # dotnet new webApp -o myWebApp --no-https
       
Test your sample application.

       # cd myWebApp
       # dotnet run
       
- Follow the instructions to access the application via a browser.
- The URL for the app is likely: http://localhost:5000
- When you are finished type Ctrl-c to stop server

## Optional: Modify the .Net code
Let's change the "Welcome" message to "Welcome from OpenShift Container Platform!".
- Navigate to the Pages directory under your project.
- In the Pages directory open the file titled Index.cshtml with your favorite editor.
- Change the following line:

      <h1 class="display-4">Welcome</h1>
      
- to:

      <h1 class="display-4">Welcome to OpenShift Container Platform!</h1>
      
- Go back to the "root" of your project folder and rerun the application to see the changed file (see previous steps)

## Prep app for OCP
I chose to deploy this example as a binary artifact.  I'll use the binary artifact with OCP to build the container that it runs. Use the following command to make the .Net app ready for the OCP build and deploy process.

      # dotnet publish myWebApp -f net5.0 -c Release
      
## Prep OCP for .Net
- Login to OCP as developer

      # eval $(crc oc-env)
      # oc login -u developer https://api.crc.testing:6443
      
- Create a new OCP project (K8s namespace)

      # oc new-project my-first-app
      
- You can check the project you are currently in with the following oc command:

      # oc status
      
- If this is your first .Net project in CRC, then you'll need to add a .Net imagestreams

      # oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
      # oc replace -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
      
## Build, Deploy and access your new .Net app
- Create a new binary build

      # oc new-build --name=my-web-app dotnet:5.0-ubi8 --binary=true
      
- Start the build and specify the path to the binary artifacts in .Net project (I'm at the "root" of my project folder).

      # oc start-build my-web-app --from-dir=bin/Release/net5.0/publish
      
- You can check the logs to see if the build is completed.

      # oc logs -f bc/my-web-app
      
- Create a new application

      # oc new-app my-web-app
      
- Make the app available to the outside world

      # oc expose service/my-web-app
      
- Get the the URL to your app

      # oc status
      
## Access the OCP console to see your project and app
- My URL looked like this: https://console-openshift-console.apps-crc.testing

      
      
