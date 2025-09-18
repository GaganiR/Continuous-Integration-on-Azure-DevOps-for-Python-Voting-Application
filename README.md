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










