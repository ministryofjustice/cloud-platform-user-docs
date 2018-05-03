---
category: cloud-platform
expires: 2018-01-31
---

# Deploying with Helm

## Overview

Helm is a Kubernetes Package Manager that can simplify the process of deploying and managing applications in the Cloud Platform.

This guide will run through the process of creating a Helm chart for an application and then deploying it into the Cloud Platform.

## Prerequisites

This guide assumes the following:

* Kubectl is installed and configured.
* Authentication with the cluster has been established.
* An environment has been created within the Cloud Platform.
* You have the deployment files for your application.

## Installing Helm

Open a new Terminal window and run the following [Brew](https://brew.sh/) command:

`brew install kubernetes-helm`

Initiate Helm. This will make the necessary local configurations and install Tiller on your cluster, in the default namespace:

`helm init`

You will now be able to use Helm to deploy an application to the Cloud Platform.

## Building the Helm chart



## Deploying the Helm chart
