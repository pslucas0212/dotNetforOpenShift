# Run .Net code on Red Hat OpenShift Container Platform on Mac OS
Create a simple Hello World .Net 5 application and run it on Red Hat OpenShift (Code Ready Containers example)
- Note: This tutorial was last updated 05 April 2021

### Pre-req .Net 5 SDK
- Note: I'm using a Mac for this example.
- Download and install [.Net 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0) for your OS.
- Note: For this tutorial I used Microsoft's installation package for SDK 5.0.203

![Download .Net 5 SDK](/images/dot02.png)

- Install the .Net SDK per the instructions for your OS
- Test your .Net installation

      % dotnet --version
      5.0.23

## Install Red Hat CodeReady Containers
- Download the Red Hat CodeReady Container (CRC) for you OS here [Create an OpenShift cluster](https://cloud.redhat.com/openshift/create/local) 
- Chose the "local" tab and select your OS.
- After you download CRC, click the Download pull secret

![Create an OpenShift cluster](/images/dot01a.png)

- After you download the code run the installer.  The latest release of CRC takes care of installing CRC for you.

## Prep your CodeReady Containers environment
- After installing CRC, we will do the rest of the work on the command line.  On the Mac open a Terminal window.  I place the pull secret in my home Documents folder in a folder I labeled crc.  On the Mac or Linux it would like this: ~/Documents/crc

      % cp pull-secret\(1\) ~/Documents/crc/pull-secret.txt

- Check the CRC version.  As of 5/18/2021 the latest release was 1.26 with OpenShift 4.7.9

      % crc version
      
- Run the setup command to download the CRC bundle and prep your environment. This typically takes a few mintues.
      
      % crc setup
        
## Start up CRC
- Start up crc an include the pull secret you previously downloaded

        % crc start -p ~/Documents/crc/pull-secret.txt

- Depending on your hardware, it make several minutes for CRC to startup.  Be sure to copy the login information that is displayed on the console after starting CRC.  See example output below.

```
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: TDjTx-BwDqi-vDvqA-VAxJz

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443
```

## Create a sample Hello World .Net web app
Now let's create our sample .Net application.  Chose a directory wher you would like to store your sample appication.  I created my sample Hello World .Net web app in a directory called projects

       % dotnet new webApp -o myWebApp --no-https
       
- When the sample app code has finished generating change into the applicaiton's directory and start up the applicaiton.

       % cd myWebApp
       % dotnet run
       
- The app starts up quickly and is ready to test when you see the last line that has the **Contentr root path:...**.  Follow the instructions to access the application via a browser.
- The URL for the app is likely: http://localhost:5000

![.Net App Welcome Page](/images/dot03.png)

- When you are finished in the terminal window on the command line type Ctrl-c to stop server

## Modify the .Net code
Let's change the "Welcome" message to "Welcome from OpenShift Container Platform!".
- Navigate to the Pages directory under your project.

      % cd Pages

- In the Pages directory open the file titled Index.cshtml with your favorite editor. 

      % nano Index.cshtml
      
- Change the following line:

      <h1 class="display-4">Welcome</h1>
      
- to:

      <h1 class="display-4">Welcome to OpenShift Container Platform!</h1>
      
- My file looked like this...

```
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div class="text-center">
    <h1 class="display-4">Welcome to OpenShift Container Platform</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>
```
      
- Go back to the "root" of your project folder and rerun the application to see the changed message

      % cd ..
      % dotnet run

![.Net Welcome to OpenShift Container Platform](/images/dot04.png)

- When finished on the command line type Ctrl-c to stop server

## Prep app for OCP
- Use the following command to make the .Net app ready for the OCP build and deploy process. The dotnet publish command preps the applicaiton for deployment storing the artifacts in a folder.  The -f swith sets the framework which .Net 5.0 in this case and -c switch defines the build configuration

      % cd ..
      % dotnet publish myWebApp -f net5.0 -c Release
     
- You are ready to go when that All projcets are up-to-date....

## Prep OCP for .Net
- Login to OCP as developer

      % eval $(crc oc-env)
      % oc login -u developer https://api.crc.testing:6443

- To see who you are logged in as type...

      % oc whoami
      developer
      
- Create a new OCP project (K8s namespace) for our .Net Welcome applicationoc

      % oc new-project my-first-app
      
- You can check the project you are currently in with the following oc command:

      % oc get projects
      NAME           DISPLAY NAME   STATUS
      my-first-app                  Active

- If this is your first .Net project in CRC, then you'll need to add a .Net imagestreams.  Imagestreams makes it easy to build and deploy our .Net app in a container and can be used to trigger new deployments when a new image becomes available   See links below for more informaton on Imagestreams

      % oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
      % oc replace -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
      
## Build, Deploy and access your new .Net app
- Create a new build configuration.  We will use a Red Hat Universal Base Image (ubi8) that includes the .Net 5 SDK and runtimes.  UBI containers are OCI-compliant.  We are building the app from binary contents.  

      % oc new-build --name=my-web-app dotnet:5.0-ubi8 --binary=true
```      
--> Found image b264294 (13 days old) in image stream "my-first-app/dotnet" under tag "5.0-ubi8" for "dotnet:5.0-ubi8"

    .NET 5 
    ------ 
    Platform for building and running .NET 5 applications

    Tags: builder, .net, dotnet, dotnetcore, dotnet-50

    * A source build using binary input will be created
      * The resulting image will be pushed to image stream tag "my-web-app:latest"
      * A binary build was created, use 'oc start-build --from-dir' to trigger a new build

--> Creating resources with label build=my-web-app ...
    imagestream.image.openshift.io "my-web-app" created
    buildconfig.build.openshift.io "my-web-app" created
--> Success
```
      
- Start the build and specify the path to the binary artifacts in .Net project (I'm at the "root" of my project folder myWebApp).

      % oc start-build my-web-app --from-dir=bin/Release/net5.0/publish
      
```
Uploading directory "bin/Release/net5.0/publish" as binary input for the build ...
...........
Uploading finished
build.build.openshift.io/my-web-app-1 started
```
      
- You can also check the logs to see if the build is completed.

      % oc logs -f bc/my-web-app
      
- Create our new application

      % oc new-app my-web-app
   
```
--> Found image 4dff9a4 (10 minutes old) in image stream "my-first-app/my-web-app" under tag "latest" for "my-web-app"

    .NET 5 
    ------ 
    Platform for building and running .NET 5 applications

    Tags: builder, .net, dotnet, dotnetcore, dotnet-50


--> Creating resources ...
    deployment.apps "my-web-app" created
    service "my-web-app" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/my-web-app' 
    Run 'oc status' to view your app.
```

- You can check the status of your app

      % oc status
   
```
In project my-first-app on server https://api.crc.testing:6443

svc/my-web-app - 10.217.4.30:8080
  deployment/my-web-app deploys istag/my-web-app:latest <-
    bc/my-web-app source builds uploaded code on istag/dotnet:5.0-ubi8 
    deployment #2 running for 46 seconds - 1 pod
    deployment #1 deployed 46 seconds ago


1 info identified, use 'oc status --suggest' to see details.
```

- At this point we can log into the OCP console and see our application

![OCP Console login](/images/dot05.png)

- Let's click on ...url

![OCP Console Topology](/images/dot06.png)

- Hover over the app icon and click on the blue arrow to see more detail about our application.  Notice there is no route defined to get to our application.  A services is generated when we created a new application, but we need to expose that service via a route to access our application from outside the OCP cluster

![application detail](/images/dot07.png)


- Back at the command line let's make our app available to the outside world

      % oc expose service/my-web-app
      route.route.openshift.io/my-web-app exposed
      
- Get the the URL to your app

      # oc status

```
In project my-first-app on server https://api.crc.testing:6443

http://my-web-app-my-first-app.apps-crc.testing to pod port 8080-tcp (svc/my-web-app)
  deployment/my-web-app deploys istag/my-web-app:latest <-
    bc/my-web-app source builds uploaded code on istag/dotnet:5.0-ubi8 
    deployment #2 running for 8 minutes - 1 pod
    deployment #1 deployed 8 minutes ago


1 info identified, use 'oc status --suggest' to see details.
```

- Back at the OCP console we now see our app is available to the outside world

![App route available](/images/dot08.png)

## Access the OCP console to see your project and app
- No we can see our application running on OCP.  My URL looked like this: https://console-openshift-console.apps-crc.testing

![.Net App Running on OCP](/images/dot09.png)

##
## References
- [GETTING STARTED WITH .NET ON RHEL 8](https://access.redhat.com/documentation/en-us/net/5.0/html-single/getting_started_with_.net_on_rhel_8/index#publishing-apps-using-dotnet_using-dotnet-on-rhel)
- [Getting Strated Guide - CodeReady Containers 1.24](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.24/html/getting_started_guide/index)
- [Managing Imagestreams OCP 4.7](https://docs.openshift.com/container-platform/4.7/openshift_images/image-streams-manage.html)
- ![.NET 5.0 SDK and Runtime - Universal Base Image](https://catalog.redhat.com/software/containers/ubi8/dotnet-50/5f62770118e80cdc21edf226?gti-tabs=unauthenticated)
