---
category: cloud-platform
expires: 2018-01-31
---
# Deploying an application to the Cloud Platform

## Overview

The aim of this guide is to walkthrough the process of deploying an application into the Cloud Platform.

This guide uses a pre-configured application as an example of how to deploy your own.

## Prerequisites

This guide assumes the following:

* Kubectl is installed and configured.
* Docker is installed and configured.
* Authentication with the cluster has been established.
* An environment has been created within the Cloud Platform.
* Access to AWS, with ECR upload permissions.
* AWS CLI configured with account credentials.

If you are not deploying your own application and would like to deploy the example application, clone the following repo:

[https://github.com/ministryofjustice/cloud-platform-demo-app](https://github.com/ministryofjustice/cloud-platform-demo-app)

## Pushing application to ECR

To deploy an application to the Cloud Platform, firstly the application image needs to be retrievable from a repository.

Amazon's ECR, within ECS is where all of the application images used by the Cloud Platform are stored.

**Creating or choosing a repository**

On the ECR start page you will have the option create a new repository, or to search for an existing one.

Create a new repository, and name it something relevant:

![Image](images/create-ecr-repo.png)

After confirmation that the repository was successfully created, ignore the list of commands and click 'Done', at the bottom of the page.

**Authenticating with the repository**

Select your newly created repository from the displayed list.

Within the repository, you will see the following section with values unique to you:

![Image](images/repo-values.png)

Click the 'View Push Commands' button.

You will then be presented with a list of preconfigured terminal commands for you to run.

Copy the first command. You will need to add `--profile yourAWSProfile` if the AWS account you're pushing to is not the default in your CLI:

`aws ecr get-login --profile mojdsd --no-include-email --region eu-west-1`

Execute the command, then proceed to execute the Docker login command provided in the Terminal.

'Login Succeeded' will confirm you have been authenticated with the repository.

**Pushing the Docker image to the repository**

Ensure the Docker image for your application has been built and is stored locally on your machine.

Now we need to tag the image with the tag provided by ECR, so I can be pushed into the correct repository.

View the 'Push Commands' again, and modify the fourth command provided, ensuring the first tag is the one currently used on your machine:

`docker tag cloud-platform-demo-app:latest 926803513772.dkr.ecr.us-west-1.amazonaws.com/cloud-platform-demo-app:latest`

Finish by running the fifth command provided, to push the image to your repository.

`docker push 926803513772.dkr.ecr.us-west-1.amazonaws.com/cloud-platform-demo-app:latest`

A quick refresh of ECR, and your image should now be displayed.

## Configuring deployment files

### Deployment
### Service
### Ingress

## Deploying application to the cluster

## Interacting with the application
