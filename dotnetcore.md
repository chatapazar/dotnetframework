# .NET Core on Kubernetes
<!-- TOC -->

- [.NET Core on Kubernetes](#net-core-on-kubernetes)
  - [Prerequisites](#prerequisites)
  - [Deploy .NET Core from Developer Console with S2I](#deploy-net-core-from-developer-console-with-s2i)
  - [Deploy .NET Core from Dockerfile with S2I](#deploy-net-core-from-dockerfile-with-s2i)
  - [Deploy .NET Core from oc command with S2I](#deploy-net-core-from-oc-command-with-s2i)
  - [Optional: Deploy .NET Core with odo](#optional-deploy-net-core-with-odo)
- [Use git to check out the .NET Core application](#use-git-to-check-out-the-net-core-application)
- [Create a new OpenShift project](#create-a-new-openshift-project)
- [Add a component for the .NET Core application](#add-a-component-for-the-net-core-application)
- [Make the .NET Core application accessible externally](#make-the-net-core-application-accessible-externally)
- [Deploy the application](#deploy-the-application)
  - [Openshift Pipelines .NET Core Support (tekton)](#openshift-pipelines-net-core-support-tekton)

<!-- /TOC -->
## Prerequisites

- install dotnet core imagestreams
~~~sh
$ oc project openshift
$ oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json

#or 

$ oc replace -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
~~~

check .net core 5 ready to use
~~~sh
$ oc describe is dotnet
~~~

- install OpenShift Pipelines Operator

## Deploy .NET Core from Developer Console with S2I

- go to Developer Console
- new project, set name : dotnetdemo1
- deploy from git, click add, select From Git
  - git url: https://github.com/redhat-developer/s2i-dotnetcore-ex
  - branche: dotnet-5.0
  - context-dir: /app
  - switch and show builder image 5 to 3 and back to 5
  - select create route
  - click create
- view output deployment, build, pod, service, route

## Deploy .NET Core from Dockerfile with S2I

- go to Developer Console
- new project, set name : dotnetdocker
- deploy from Dockerfile, click add, select From Dockerfile
  - git url: https://github.com/chatapazar/qotd-csharp
  - show Dockerfile --> github 
    ~~~docker
    FROM registry.access.redhat.com/ubi8/dotnet-31:3.1
    USER 1001
    RUN mkdir qotd-csharp
    WORKDIR qotd-csharp
    ADD . .

    RUN dotnet publish -c Release

    EXPOSE 8080

    CMD ["dotnet", "./bin/Release/netcoreapp3.0/publish/qotd-csharp.dll"]
    ~~~

## Deploy .NET Core from oc command with S2I

- Create a new OpenShift project
    ~~~sh
    $ oc new-project dotnetdemo2
    #Add the .NET Core application
    $ oc new-app dotnet:5.0-ubi8~https://github.com/redhat-developer/s2i-dotnetcore-ex#dotnet-5.0 --context-dir app

    #Make the .NET Core application accessible externally and show the url
    $ oc expose service s2i-dotnetcore-ex
    $ oc get deployment,pods,svc,route
    ~~~

## Optional: Deploy .NET Core with odo

~~~sh
# Use git to check out the .NET Core application
$ git clone https://github.com/redhat-developer/s2i-dotnetcore-ex
$ cd s2i-dotnetcore-ex/app
$ git checkout dotnet-5.0

# Create a new OpenShift project
$ odo project create mydemo

# Add a component for the .NET Core application
$ odo create dotnet:5.0-ubi8

# Make the .NET Core application accessible externally
$ odo url create

# Deploy the application
$ odo push
~~~

## Openshift Pipelines .NET Core Support (tekton)

- install OpenShift Pipelines Operator
- show demo git url: https://github.com/redhat-developer/s2i-dotnetcore-ex/tree/dotnetcore-3.1-openshift-manual-pipeline
- demo command
~~~sh
$ cd s2i-dotnetcore-ex
$ git checkout dotnetcore-3.1-openshift-manual-pipeline
#create project
$ oc new-project dotnet-pipeline-app
#create task,pipeline,resource
$ oc apply -f ci/simple/simple-pipeline.yaml
#show pipelines in dotnet-pipeline-app (developer console)
#create pipelinerun
$ oc create -f ci/simple/run-simple-pipeline.yaml
#show pipelinerun, view log, show build and test
~~~



