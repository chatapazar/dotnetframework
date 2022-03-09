# .NET Framework on K8S
<!-- TOC -->

- [.NET Framework on K8S](#net-framework-on-k8s)
  - [Prerequisites](#prerequisites)
  - [Review Windows Container Node](#review-windows-container-node)
  - [Build .NET Framework](#build-net-framework)
  - [Build .NET Framework Windows Container (Build Code with Visual Stuidio and Build image with Docker)](#build-net-framework-windows-container-build-code-with-visual-stuidio-and-build-image-with-docker)
  - [Build .NET Framework Windows Container (Build code in Docker on your laptop)](#build-net-framework-windows-container-build-code-in-docker-on-your-laptop)

<!-- /TOC -->
## Prerequisites

1. Request RHPDS --> Service "OpenShift 4.6 Windows Node Demo"
2. Create Nexus
    - follow instruction at https://github.com/rhthsa/openshift-demo/blob/main/ci-cd-with-jenkins.md
    - redeploy if it not success, becouse deploy wrong node --> windows
    - set new password admin/admin
    - set allow anonymous access
3. Prepull image with docker desktop for windows
   - Instruction http://people.redhat.com/chernand/windows-containers-quickstart/predeploy-steps/
      ~~~sh
      $ docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019 
      ~~~
4. build & push image to nexus before demo (case networkslow)
      ~~~sh
      $ docker login nexus-registry-ci-cd.apps.cluster-xxx.opentlc.com
      #input user/password admin/admin
      $ docker push nexus-registry-ci-cd.apps.cluster-xxx.opentlc.com/aspnetapp:4.7.2
      ~~~
5. clone git to windows workstation for demo, git url: https://github.com/chatapazar/dotnetframework.git
   - sample folder
      ~~~sh
      F:\workspace\dotnetframework
      ~~~
6. Developer Laptop OS: Windows 10 Pro
7. Install Docker Desktop for Windows, set to Windows Container
8. Pull local Docker base image mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
9. Install OC CLI for windows
10. Install Visual Studio 2019
   - Install .NET Framework 3.5, 4.7, 4.8 Based On Your Requirement
11. add nuget manage to https://packages.nuget.org/v1/FeedService.svc/
12. restore with nuget manager in visual studio before demo

## Review Windows Container Node

- login to openshift console
- go to node, view linux node, view windows node
- view OS, OS Image, Container Runtimes

## Build .NET Framework

- open visual studio 
  - Solution Path F:\workspace\dotnetframework\WebApplication1\WebApplication1.sln
  - click show all file in solution explorer

- new sample project (Don't create, show only)
- show .net framework support image at https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/

- show project version 3.5, 4.x
- select WebApplication2 Project, run WebApplication2 version 4.7
- set startup project, right click project set statup
- click run IIS Express, review, close browser

## Build .NET Framework Windows Container (Build Code with Visual Stuidio and Build image with Docker)

- switch docker to windows container mode before build and run this step!!!
- click publish at project, publish to folder --> click publish
- publish to docker hub (option) https://docs.microsoft.com/en-us/visualstudio/containers/deploy-docker-hub?view=vs-2019
- login to openshift (with windows node)
  ~~~sh
  $ oc login
  #user admin/r3dh4t1
  ~~~
- show windows node-->machine->windows , view node from this machine-> show os image
- start docker desktop, check windows containers
- show base image --> mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
- show Dockerfile in visual studio (select show all file in solution explorer)
  - Review Dockerfile
- go to WebApplication2 command line
- test docker run with docker dashboard set port 8080
  ```sh
  #Create Docker Image
  $ cd WebApplication2
  $ docker build -t aspnetapp:4.7.2 .
  $ docker run -it --rm -p 8080:80 aspnetapp:4.7.2
  #ctrl + c for exit
  $ docker tag aspnetapp:4.7.2 nexus-registry-ci-cd.apps.cluster-xxx.xxx.sandbox1123.opentlc.com/aspnetapp:4.7.2
  $ docker login nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com
  #user/password : admin/admin
  #show push, don't push at demo time
  $ docker push nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com/aspnetapp:4.7.2
  ```
- show docker in nexus
- deploy
  ```sh
  $ oc new-project dotnetframework
  # review deploy-asp472-nexus.yaml, change nexus registry

  $ oc create secret docker-registry nexus --docker-server=nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com --docker-username=admin --docker-password=admin
  $ oc secrets link default nexus --for=pull
  $ oc secrets link deployer nexus --for=pull  
  # edit yaml change image value before run
  ###############################
  # show node selector windows
  # show taint/toleration
  ###############################
  $ oc create -f .\deploy-asp472-nexus.yaml
  $ oc expose svc aspnex472
  ```

- example taint/toleration
- show taint in windows node (openshift admin console ui)
  ```yaml
      spec:
        tolerations:
        - key: "os"
          value: "Windows"
          Effect: "NoSchedule"
        containers:
        - name: aspnex472
          image: nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com/aspnetapp:4.7.2
          imagePullPolicy: IfNotPresent        
        nodeSelector:
          beta.kubernetes.io/os: windows
  ```

## Build .NET Framework Windows Container (Build code in Docker on your laptop)

- pull sdk image to your laptop
  ```ssh
  docker pull mcr.microsoft.com/dotnet/framework/sdk:4.8
  ```
- run docker sdk & mount disk to your code (or use another way to check out source code to your container such as git)
  ```ssh
  docker run -v F:\workspace\dotnetframework\WebApplication1:c:\workspace -it --rm mcr.microsoft.com/dotnet/framework/sdk:4.8 cmd.exe 
  ```
- after shell in container, go to c:\workspace , run nuget restore at sln path and run publish
  ```cmd
  #go to sln path
  nuget restore
  #go to your project path such as go to c:\workspace\WebApplication3 , and run msbuild
  MSBuild WebApplication3.csproj /t:Rebuild /p:Configuration=Release
  ```
- prepare Dockerfile , dockerfile example is (create dockerfile at project path)
  ```Dockerfile
  FROM mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
  WORKDIR /inetpub/wwwroot
  COPY . .
  COPY ./bin ./bin
  ```
- build image
  ```ssh
  # go to project path
  docker build -t aspnetapp:4.7.2 .
  ```
- after complete build image, run test in your laptop with
  ```ssh
  docker run -it --rm -p 8080:80 aspnetapp:4.7.2
  ```