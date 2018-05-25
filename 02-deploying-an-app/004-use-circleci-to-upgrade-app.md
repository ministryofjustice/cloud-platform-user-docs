---
category: cloud-platform
expires: 2018-01-31
---

# Continuous Deployment of an application using CircleCI and Helm

## Introduction
This document covers how to continuously deploy your application in the Cloud Platform. It is essentially a continuation of [‘Deploying an application to the Cloud Platform with Helm’](https://ministryofjustice.github.io/cloud-platform-user-docs/02-deploying-an-app/001-app-deploy-helm/#tutorial-deploying-an-application-to-the-cloud-platform-with-helm). 

*Note: This document is specific to using [CircleCI](https://circleci.com/) as the Contionious Integration method.*

### Objective
By the end of the tutorial, you will have done the following:

- Created a Service Account for CircleCI in your namespace
- Generated a CircleCI configuration file in your application repository. The configuration file will authenticate with your chosen cluster, build a docker image and push it to ECR and upgrade your helm deployment with the new docker image.
- Have an automated CircleCI pipeline that upgrades your helm deployment when a new change is pushed to your master branch

### Requirements
It is assumed you have the following: 

 - You have a basic understanding of what [Kubernetes](https://kubernetes.io/) is.
 - You have [created an environment for your application](/01-getting-started/003-env-create)
 - You have installed [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) on your local machine. 
 - You have [Authenticated](/01-getting-started/002-authenticate) to the cluster known as the 'non-production cluster'.
 - You have [deployed an application](https://ministryofjustice.github.io/cloud-platform-user-docs/02-deploying-an-app/001-app-deploy-helm/#tutorial-deploying-an-application-to-the-cloud-platform-with-helm) to the 'non-production cluster' using Helm.

### Creating a Service Account for CircleCI
As part of the CircleCI deployment pipeline, CircleCI will need to authenticate with the kubernetes cluster. In order to do so, Kubernetes uses [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/). Service Accounts provide an identity for processes that run in a cluster allowing the process to access the API server.

A Service Account is created in the [namespace creation github repository](). 
