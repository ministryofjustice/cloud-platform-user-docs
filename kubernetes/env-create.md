---
category: Kubernetes
expires: 2018-01-31
---
# Creating a Kubernetes Environment

## Aim

This is a guide to creating a non-production environment in our non-production Kubernetes cluster. This document is intended for any developers who are interested in creating a development or staging environments within Kubernetes.

## Description

K8s-nonprod-environments repo : - [https://github.com/ministryofjustice/k8s-nonprod-environments|https://github.com/ministryofjustice/k8s-nonprod-environments]

This repo contains the necessary files to create a pipeline in aws to create Kubernetes cluster namespaces and resources after a push has been made to the master branch of this repo. This repo contains the following files and directories

#### Terraform

This directory contains terraform resources to create AWS pipeline for creation of kubernetes namespave and resource creation.

#### Buildspec

The buildspec.yaml file contains codebuild specification that will create resources defined in the namespaces.

#### Namespace

The namespaces contains subdirectories named after each of the desired namespaces you want to create.

You will be working within the namespace directory to create your environment.

### GitHub directory structure

### Instructions

![Image](http://images/image2.png)

1.  Git clone the repo onto your local machine and create a new branch to make your changes. (Using terminal)
CODE BLOCK

2. Now we need to amend the namespaces directory in the root of repo and create an directory within it.
CODE BLOCK

3. Now we will need to create two files within our $servicename-$env directory.  Namespaces.yaml and $servicename-$env-admin-role.yaml. The structures of both files are shown below.
TABLE CODE BLOCK

4. After both files are created. You are ready to upload your branch to our github repo. You will need to add the files, commit them and then set a new upstream branch.
TABLE CODE BLOCK

5. Now that you have uploaded your branch, go to the Github GUI and select your branch. Once you have done this, you may make a pull request to merge to master.
IMAGE
