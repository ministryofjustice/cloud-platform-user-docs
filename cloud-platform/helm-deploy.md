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

### Generating a template chart

With Helm now configure on your machine, you can start the process of creating a Helm chart template.

In a directory of your choice, issue the following command to generate a Helm chart template:

`helm create yourName` - NOTE: `yourName` will be the name of your chart, this can be changed to your liking.

Within your directory, you will now see a new sub-directory with the name of your chart.

### Helm chart structure

Within your newly generated chart, you will find 4 objects at it's root level, 2 files and 2 directories:

* `Chart.yaml` - This file contains basic information about your chart, App version, for example:

```yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: yourName
version: 0.1.0
```

* `values.yaml` - This file is where secrets or custom variables are stored, which can be later pulled into the other templates in the chart:

* `templates` - The Template directory is where you store all of the manifest files for your applications deployment. You will notice some file templates have already been populated.

* `charts` - This directory can also be known as a dependency folder. Essentially if the Helm chart you are creating depends on other Helm charts to run correctly, these charts must be stored here.

### Configuring your Helm chart

This section of the guide will walk through the process of taking the generated chart template and compile it into the chart for your application.

#### Chart.yaml

We will start with the `Chart.yaml` file, by opening in an editor:

```yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: yourName
version: 0.1.0
```

The `appVersion:` value should be changed to the version of your application.

The `description:` value should provide a short description of what application the chart will deploy.

The `name:` value should reflect the name of this chart.

The `version:` value is the version of your chart and should be updated when changes are made to the chart. This is not to be confused with the version of the application itself.

#### templates

Within the directory you will see some example files. Delete these files and replace them with the manifest files for your application, which should be:
* `deployment.yaml` - `service.yaml` - `ingress.yaml`

#### charts

You can leave this directory empty, unless your application depends on another application, in which case, add the chart within this directory.

#### values.yaml

For now, comment out the contents of this file and save the changes.

## Deploying the Helm chart

With the Helm chart configured for your application, you can now go ahead and deploy it with:

`helm install .\yourName`

Confirm the deployment was successful with:

`helm list`

You can also delete the deployment with:

`helm delete RELEASE_NAME`
