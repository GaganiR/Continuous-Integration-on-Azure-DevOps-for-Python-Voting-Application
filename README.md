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





Now we need to create three pipelines for vote, result, and worker.





