# dotnetframework
prerequisite
1. rhpds --> Service "OpenShift 4.6 Windows Node Demo"
2. nexus --> follow instruction at https://github.com/rhthsa/openshift-demo/blob/main/ci-cd-with-jenkins.md
(redeploy if it not success, becouse deploy wrong node --> windows)
https://github.com/chatapazar/dotnetframework.git
set new password admin/admin
set anonymous
3. pull mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019 with docker desktop for windows
4. docker push nexus-registry-ci-cd.apps.cluster-xxx.opentlc.com/aspnetapp:4.7.2
5. oc login
6. pre deploy image from nexus 
7. prepull image pull mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
in windows node
http://people.redhat.com/chernand/windows-containers-quickstart/predeploy-steps/



F:\workspace\dotnetframework

git clone to this path

open visual studio 
F:\workspace\dotnetframework\WebApplication1\WebApplication1.sln
click show all file in solution explorer

new project sample don't real create
show .net framework support image 
https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/

show project version 3.5, 4.x

run WebApplication2 version 4.7
set startup project
click run IIS Express, review, close browser
publish to folder --> click publish
publish to docker hub (option) https://docs.microsoft.com/en-us/visualstudio/containers/deploy-docker-hub?view=vs-2019

login to openshift (with windows node)
user admin/r3dh4t1
show windows node-->machine->windows , view node from this machine-> show os image

start docker desktop, check windows containers
show base image
show dockerfile in visual studio

go to WebApplication2 command line
docker build -t aspnetapp:4.7.2 .
tag with nexus registry , view url from openshift dev console nexus 
such as : nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com

docker tag aspnetapp:4.7.2 nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com/aspnetapp:4.7.2


docker login nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com
user/password : admin/admin

show only 
docker push nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com/aspnetapp:4.7.2

show nexus

oc new-project dotnetframework

view deploy-asp472-nexus.yaml
change nexus registry

oc create secret docker-registry nexus --docker-server=nexus-registry-ci-cd.apps.cluster-kbtg-e44a.kbtg-e44a.sandbox1123.opentlc.com --docker-username=admin --docker-password=admin
oc secrets link default nexus --for=pull
 oc secrets link deployer nexus --for=pull  
   oc create -f .\deploy-asp472-nexus.yaml
   oc expose svc aspnex472