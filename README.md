
# Managing Credentials In Kubernetes

### Method-1 With GitHub Actions Pipeline

### Architecture Diagram

![POC1GHAP drawio](https://github.com/urolimevishnu/devcicd/assets/124776856/c02f7899-f491-469e-a36d-d4f602542c25)

This task consists of creating two kubernetes clusters for two environments as one for dev and other for prod using terraform on GCP. Then we need to create a sample application with database using helm chart. For that i have created a helm chart named wordpress which is provided in this repository along with the terraform files needed for the GKE cluster deployment.

First we need to create a service account with necessary permissions and a bucket for storing the terraform state files. We also need to generate the credential for accessing the GCP by the service account we created. Then apply the terraform files for each cluster.

After that go to your github repo were you have your helm chart. create repository secrets for:
- DB_PASSWORD (password for your mysql database)
- DB_USERNAME (username for your mysql database)
- GCLOUD_AUTH (Credential for service account)
- PROJECT_ID (project id of your GCP)
- DEV_CLUSTER (name of dev cluster)
- PROD_CLUSTER (name of prod cluster)
- DEV_ZONE (dev cluster zone)
- PROD_ZONE (prod cluster zone)

And give the necessary values for them. You also need to change the values in the values.yaml file in the helm chart as your need. 

These values are used in the workflow file for the pipeline so that the application will be deployed in both of the cluster.

When any changes happens in our branch, then it will trigger and runs the pipeline automatically in the cluster through github actions.

The workflow.yaml file is also provided in this repo. You need to create directory structure like ".github/workflows/workflow.yaml" inside your repository.

I have also provided a script for backing up and restore the database of the dev environment into prod environment for seameless updates to the production webapp from dev webapp.

You will be asked to provide your project id, dev and prod cluster name, dev username and password for database, prod username and password for database, dev and prod pod name etc.



### Method-2 Without pipeline Using Kubeseal to secure Kubernetes Credentials

### Architecture Diagram

![devtask1 drawio](https://github.com/urolimevishnu/devcicd/assets/124776856/a233407f-72e2-495b-9796-322d03018cc7)


This stage consistes of creating two terraform files which will be used to create two GKE clusters. We are using the same terraform files we used earlier. Then we need to create a helm chart which we will use to create a wordpress application in the GKE clusters.

We are introducing a new tool in this method named "Kubeseal".
It is used to encrypt a data in a kubernetes cluster which can be shared with anyone without consern about security breach. 

We are using Kubeseal to encrypt the kubernetes secrets file that is given inside the Helm Chart which we will need for deploying the wordpress application. 

For that first you need to install Kubeseal in your machine. Then you need to connect to the cluster and install the kubeseal controller in that cluster then get the public and private key from the kubeseal controller and save it in a secure location.
Now you need to use the public key and encrypt the kubernetes secrets file inside the Helm Chart and use the new file that is generated inside the helmchart and remove the existing file from there. I have added the helmchart that i used in this repo, you can check it as reference you need to do this in both ckusters seperately and you need to setup kubeseal controller for both clusters seperately.

You can now share the helmchart to your friends or team mates without exposing your real username or password that is used for the database.

Now you only need to deploy it into your cluster by the "helm install" command. 

You need to setup this in both clusters manually because you don't have a pipeline in this setup for automatic deployment.

Taking backup and resore can be done using the same script we used earlier.

#### If you don't need a pipelien you can use this setup because it can be reused and all the details about your task is stored inside your cluster.


