# Continuous Integration on Azure DevOps for Python Voting Application

## Prerequisites
* Azure Devops subscription.
* Create Resource group. I named it as azure-CICD.

## Import repository to AZ DevOps
* Create a new project and name it as voting-app.

<img width="1237" height="897" alt="image" src="https://github.com/user-attachments/assets/01cac487-fc66-4e9c-b9f2-ca7e7198c6b3" />

* Now we need to import the Git project to the AZ Devops platform.
* Search for the __example-voting-app__ in the github __dockersamples__. Copy the HTTPS URL.
https://github.com/dockersamples/example-voting-app.git
* Go to Repos section in AZ DevOps.
* Inside "Import a repository", paste this link. It doesnt require authentication as it is a public repo.
  
<img width="595" height="872" alt="image" src="https://github.com/user-attachments/assets/775f327a-a05a-469a-81b0-ae4d85bdffaa" />

* When the import is successful, you'll see that the project is now on AZ Devops.
  
<img width="1717" height="700" alt="image" src="https://github.com/user-attachments/assets/5c25d6c7-5287-40a2-8ac5-93bc57a6ecc2" />

* Notice at the top that the default branch is not main. It will be some other branch which AZ Devops chooses in default alphabetical order.
* Change it to main by going to Repos> Branches.
* Near the main branch, click the three dots and set main as default branch.
  
<img width="1583" height="720" alt="image" src="https://github.com/user-attachments/assets/13e40451-a175-4c50-94bc-82476576a9c0" />
<img width="836" height="726" alt="image" src="https://github.com/user-attachments/assets/a3c19c89-ca5d-4588-85bf-1bad28fa7a6c" />


## Create Container Registry
The docker containers we create for the vote, results and worker will be pushed to this registry.

* Now go to the container registries on Microsoft Azure and create a new registry named gaganiazure.
  
<img width="1135" height="580" alt="image" src="https://github.com/user-attachments/assets/1daf84be-3375-428b-ae02-5f5212f8977a" />

## Create Pipelines
Now we need to create three pipelines for vote, result, and worker.

* On the Azure DevOps Platform go to the pipelines section.
* Click on reate a pipeline

<img width="1421" height="772" alt="image" src="https://github.com/user-attachments/assets/23a56c35-bf13-44d3-a15f-150450523529" />

* Select azure repos git for source code.
* Select your previously created repo (voting-app)
  
<img width="887" height="282" alt="image" src="https://github.com/user-attachments/assets/991d42d3-8793-49f2-8a39-e4898727c16c" />

* In the docker options select "Build and push an image to Azure Container Registry".
* Choose the registry and give a name for the image. can change these later as well.
  
<img width="745" height="871" alt="image" src="https://github.com/user-attachments/assets/d3b57b26-eb74-4710-8e7c-e9914de3eea0" />

* Since we selected a template for the pipeline, this will give us the structure of the code that we can customize accordingly.
* First we will create a pipeline for the __results__ microservice.

## Result Microservice Pipeline
* Change the file name at the top "azure-pipelines.yml" to "azure-pipelines-result.yml".
  
<img width="520" height="236" alt="image" src="https://github.com/user-attachments/assets/6a91bb85-53be-4f5a-92d7-fe89bd8f4cdc" />
<img width="562" height="233" alt="image" src="https://github.com/user-attachments/assets/57ee0717-e0fa-43cf-a33a-820d1225c930" />

* Make Changes to the trigger.
* We will use a path based trigger for the pipeline to trigger its build when code change is done to a particular path which is the result microservice.
* To trigger on every change under result directory, make the following code changes.
```
trigger:
  paths:
    include:
      - result/*
```
<img width="378" height="237" alt="image" src="https://github.com/user-attachments/assets/fa286919-31f2-4ac4-bb0a-642fc09033ea" />

* Below that we get some variables that are populated automatically.
  
<img width="787" height="185" alt="image" src="https://github.com/user-attachments/assets/be4def2e-b975-42b6-9b11-91737af9df6f" />

* __dockerRegistryServiceConnection__ - ceates connection between az devops platform and your az container registry on the az platform.
* __imageRepository__ - repo for image within container registry
* __containerRegistry__ - is same for all the microservices in a particular project
* __dockerfilePath__ - path of dockerfile for each microservice
* __tag__ - tags image with build number.
* change __imageRepository__ from votingapp to resultapp.
  
<img width="780" height="175" alt="image" src="https://github.com/user-attachments/assets/5b3627ab-d1f0-423f-8e44-dd8c8ced48bf" />

* Next we need to configure an agent for entire CI process to run.
* For example in Jenkins, you need to create your own slave/worker nodes and you make your Jenkins pipelines to run on it.
* Similarly for azure pipelines, there are two options.
* 1) Can create your own agent and run your pipeline on top of that agent.
  2) Can use the agent that azure provides automatically. However this feature is not available in teh free tier.
* Threrefore we need to create a pool and give a name for the agent.
* Then we need to create a VM in the az portal. This VM will act as an agent for our CI Pipelines. Our pipelines in az Devops platform will run on the VM agent in the az Portal.
Change this:

<img width="321" height="72" alt="image" src="https://github.com/user-attachments/assets/006c5892-32a4-49da-90b8-150bbb83f428" />

to this:

<img width="256" height="102" alt="image" src="https://github.com/user-attachments/assets/0bd4e237-d466-4ac3-a40b-94be291851a9" />

* In the __Stages__ section, the template gives build and push in a single section. Bute we will split it into two.
* Delete the push words and leave everything in that stage as build.
* Delete the part with the pool.
* Click on settings near the tasks and change command to build only. not build and push.
  
<img width="632" height="458" alt="image" src="https://github.com/user-attachments/assets/545c853f-8dc2-444c-851a-5b3e8115bf85" />



<img width="427" height="257" alt="image" src="https://github.com/user-attachments/assets/eb80d885-6f67-45df-8260-5aa781cd2061" />

* Now create the push stage.
<img width="632" height="403" alt="image" src="https://github.com/user-attachments/assets/aa0f6e49-3831-4498-b28f-c70bfcfd1086" />

* Complete yaml file will look like this now:
```
# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
  paths:
    include:
      - result/* 


resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: '5251a488-d739-4a97-9e3a-3daccafdacfd'
  imageRepository: 'resultapp'
  containerRegistry: 'gaganiazure.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
  tag: '$(Build.BuildId)'

  # Agent VM image name
pool:
  name: 'azureagent'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    
    steps:
    - task: Docker@2
      displayName: Build 
    
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: 'result/Dockerfile'
        tags: '$(tag)'
- stage: Push
  displayName: Push stage
  jobs:
  - job: Push
    displayName: Push
    
    steps:
    - task: Docker@2
      displayName: Push 
    
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'push'
        tags: '$(tag)'
```
* Now click on __save and run__ button to try building the pipeline.
* It should return an error message saying there is no agent.
<img width="1057" height="177" alt="image" src="https://github.com/user-attachments/assets/65ba5792-2113-46b3-b0e6-71b58e8447a4" />

* This is because we have not yet created a VM and assigned it as an agent even we've only mentioned that we will be using an agent.

## Create VM Agent
* create a VM in the sam resource group.
* In az devops portal go to project settings>agent pools>add pool.
* In the configuration, select self-hosted and name it as azureagent. tick the box Grant access permission to all pipelines. click create.
  
<img width="605" height="862" alt="image" src="https://github.com/user-attachments/assets/55ee18bf-0a01-4203-9be5-2a40a44b963e" />
 <img width="387" height="385" alt="image" src="https://github.com/user-attachments/assets/1e467702-21f5-4709-9ffb-0f9d08a96383" />

* to integrate this VM called azureagent which is on the Microsoft Azure platform with the pipelines that are on the Azure DevOps Platform.

For this, there are some commands which you will get from Azure DevOps documentation.

Link For Azure DevOps Docs:https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops
Direct Link to Self-hosted Linux agents:https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops
* select the created pool. click on agents tab > new agent.
* in the tab open select linux. we need to run these commands on our VM.
  
<img width="982" height="820" alt="image" src="https://github.com/user-attachments/assets/a3580ec4-749b-479b-9883-4347e9d1cd7a" />

* Open gitbash.
* move to downloads folder.
* give permissions to .pem file
* ssh into the VM.
* copy the commands from the previous agent tab and run on the terminal.
* move into myagent folder
```
mkdir myagent && cd myagent
```
* first copy that link and use wget to download.
```
 wget https://download.agent.dev.azure.com/agent/4.261.0/vsts-agent-linux-x64-4.261.0.tar.gz
```
* run the tar file
```
tar zxvf vsts-agent-linux-x64-4.261.0.tar.gz
```
then run ./config.sh

it asks for server URL. get it from the azure docs.

<img width="620" height="145" alt="image" src="https://github.com/user-attachments/assets/07d97d9c-c907-4ad4-bbfc-30d8b26f2848" />

replace {your-organization} with the name of your org registered in your azure devops platform.

 <img width="445" height="178" alt="image" src="https://github.com/user-attachments/assets/92ba20ea-e621-4e4a-bd79-a2179ab85a7f" />

it will ask for PAT. Get it from azure devops platform > user settings > personal access token. give a name and create.

<img width="777" height="862" alt="image" src="https://github.com/user-attachments/assets/a27aae95-159c-47e0-897a-8b7a56c03679" />

copy the displayed PAT and paste it in the command.

it will prompt you to enter the azure agent pool name. my created pool name is azureagent.

now the agent connection is successful.

To verify the azure agent, go to azure devops and go to settings>agents.

You will see that the agent is created but its offline. 

<img width="913" height="302" alt="image" src="https://github.com/user-attachments/assets/1e9dfea3-edae-45b0-8dd8-3418d7f1d6ac" />

Before we turn the agent online or run jobs, we need to install docker on the agent since we're running containers.

```
sudo apt install docker.io
```
Grant the azureuser permissions to the docker daemon.
```
sudo usermod -aG docker azureuser
sudo systemctl restart docker
sudo systemctl status docker
```
<img width="691" height="106" alt="image" src="https://github.com/user-attachments/assets/59b263d0-a8e4-4eaf-839b-5b9f9e5de5cb" />

Now Start the agent using 
```
./run.sh
```
<img width="401" height="90" alt="image" src="https://github.com/user-attachments/assets/371482be-4add-4518-85ff-734f7b024cd7" />

Now the agent is online

<img width="653" height="292" alt="image" src="https://github.com/user-attachments/assets/de6a9cb4-dadc-448c-8e40-4d2a724478b6" />

On azure devops, go to pipeline>run pipeline.

<img width="738" height="773" alt="image" src="https://github.com/user-attachments/assets/0cf8252c-89b8-4bf1-9410-789462f3950b" />

<img width="712" height="382" alt="image" src="https://github.com/user-attachments/assets/a8202530-15ed-4831-a9fb-6bbf09a6beaa" />

<img width="658" height="232" alt="image" src="https://github.com/user-attachments/assets/aa6de1fd-afed-4004-8ed1-d57842b1997e" />

now try doing a change to the code of result yaml. when committed, the pipeline should run automatically. I entered a comment to the code.

<img width="395" height="205" alt="image" src="https://github.com/user-attachments/assets/1d8c93e0-0e9b-4213-b630-0669705b3049" />

Notice in the pipeline running, now it says "individual CI" instead of "manually run"

<img width="1513" height="456" alt="image" src="https://github.com/user-attachments/assets/d23304ff-b9bd-4bf0-9ad9-b3b0fe097a79" />

Make the same changes to the vote and worker microservice as well. there are two pipelines created now with the same name voting-app. rename the first one as result-service and second one as vote-service.

<img width="1376" height="482" alt="image" src="https://github.com/user-attachments/assets/57391f7d-fd0b-4364-bd5e-76301cd23754" />






