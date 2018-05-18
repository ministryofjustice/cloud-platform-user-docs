---
category: cloud-platform
expires: 2018-01-31
---
# Tutorial: Deploying an application to the Cloud Platform with Helm

## Introduction
This document will act as a guide to your first application deployment into the Cloud Platform. If you have any issues completing the objective or have any suggestions please feel free to drop use a line in the #cloud-platform-users slack channel. 
### Objective
By the end of this guide you'll have deployed a reference [Django application](https://github.com/ministryofjustice/cloud-platform-reference-app) to the Non-production cluster using the Kubernetes package manager [Helm](https://helm.sh/).

*Disclaimer: You'll see fairly quickly that the application is not fit for production. A perfect example of this is the [plaintext secrets file](https://github.com/ministryofjustice/cloud-platform-reference-app/blob/master/helm_deploy/django-app/templates/secret.yaml). For the reference application we've left this file in plaintext but it **must** be encrypted when writing your own manifests for production/non-production work in the MoJ.*

### Requirements
It is assumed you have the following: 

 - You have a basic understanding of what [Kubernetes](https://kubernetes.io/) is.
 - You have [created an environment for your application](/cloud-platform/env-create)
 - [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) is installed on your local machine.
 - [Docker](https://docs.docker.com/install/) is installed on your local machine. 
 - [Authentication](/cloud-platform/authenticate) has been established with the Non-production cluster.

## Deploy the app
The reference application we're going to use is a very simple Django application with an on-cluster Postgresql database.

> Note: Even though we are going to install a database within the Kubernetes cluster, it is recommended to use a database as a service offering such as [AWS RDS](https://aws.amazon.com/rds/) if running in production. 

Lucky for you the Helm deployment manifests have been pre-written for this exercise. But if you wish to know more about these files and what they do have a quick browse of the [README](https://github.com/ministryofjustice/cloud-platform-reference-app/tree/master/helm_deploy/django-app/README.md).

### Set up
First and foremost we need to clone our reference application and subsequently change directory: 

    $ git clone https://github.com/ministryofjustice/cloud-platform-reference-app.git
    $ cd cloud-platform-reference-app

You now have a functioning git repository that contains a Django simple application. Have a browse around and get familiar with the directory structure.

### Nothing up my sleeves!
Let's make use of the command line tool `kubectl` to browse around the cluster to see what it looks like before we deploy our application: 

    $ kubectl get pods -namespace <env-name>
*The `<env-name>` here is the environment you created, listed in the requirements section at the beginning of the document.* 

You should be met with a simple response of: 

    No resources found.

If you receive the below error message then you've either not typed in your environment name correctly or you don't have permission to perform a `get pods` command. Either way, you'll need to go back and review the [Creating an Environment](https://ministryofjustice.github.io/cloud-platform-user-docs/cloud-platform/env-create/#creating-a-cloud-platform-environment) document previously mentioned. 

    $ Error from server (Forbidden): pods is forbidden: User "test-user" cannot list pods in the namespace "demo"

### Using Helm
First, what is Helm? Helm allows you to manage application deployment to Kubernetes using Charts. Have a quick read of some of the many features of Helm Charts [here](https://docs.helm.sh/developing_charts/). 

I can tell you're excited to get started but we're just going to focus on the deployment of a Helm Chart in this guide. We'll link/create further guides on how to take full advantage of the awesome features available in Helm in the 'Next Steps' below. 

Now it's time to get our hands dirty. Let's deploy the application.

First, change directory so we can focus on the app we need:

    $ cd helm_deploy/django-app

Values for our application are stored in the aptly named `values.yaml` at the root of our directory. There is a single value we need to change before we deploy the application. This is the host name, which denotes the URL of your deployed application. Open this file with your favourite text editor and have a look. 

The value to which I refer is:

    deploy:
      hosts: helm-demo.apps.cloud-platforms-test.k8s.integration.dsd.io

Keeping this value the same would deploy your application using this URL and would more than likely cause a conflict. Fear not. We can change this value by setting it as an argument in our command line install. 

Run the following (replacing the `YourName` with your own name and `env-name` with your environment name:

        $ helm install . \
          --name django-app-<YourName> \
          --namespace <env-name> \
          --set deploy.hosts=django-<YourName>.apps.cloud-platforms-test.k8s.integration.dsd.io

The `set deploy.hosts` overwrites the value stored in my value.yaml file and you'll see a fairly verbose output showing your pods creating. 
### Viewing your application
Congratulations on getting this far. If all went your pods are now deployed and is now being served on your specified URL. 

Let's check:

    $ kubectl get pods -namespace <env-name>

If the deploy was successful you should be greeted with something similar to the below:

```
NAME                                           READY     STATUS    RESTARTS   AGE
django-app-<Name>-fcc657679-w69cr               1/1       Running   1          39m
django-app-<Name>-postgresql-7b4bdff4b8-xdlw2   1/1       Running   0          39m
```
You should have a postgres and app pod **ready** with the status **running**.

Let's check your host has a URL by running:

    $ kubectl get ingress -namespace <env-name>

This will return the URL of your given app. Open it using your favourite browser. 

You should be met with an MoJ reference app with the title, *'Cloud Platforms Deployment'*. As we mentioned before, there is nothing complicated about this application. You can enter your name and job role, calling the on-cluster postgresql database. 

### View the logs
Each pod will generate logs that can be viewed via the API. Let's have a browse of our application logs.

First grab the pod name: 

    $ kubectl get pods -namespace <env-name>

Then copy the pod name that isn't postgresql.

 ```
   NAME                                           READY     STATUS    RESTARTS   AGE
* django-app-<name>-fcc657679-w69cr               1/1       Running   1          54m
   django-app-<name>-postgresql-7b4bdff4b8-xdlw2   1/1       Running   0          54m
```

We're going to follow the log, so we'll run:

`kubectl log django-app-<name>-fcc657679-w69cr -namespace <env-name> -f`

As you can see, this tails the log nicely and you should see our health checks giving a HTTP 200. 

Read more about Kubernetes logging [here](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

### Scale the application
You now have our application up and running but you decide one pod isn't enough. Say you want to run three. This is simply a case of changing the replicaCount value in the values.yaml whilst running the `helm upgrade` command.

Let's try:

    $ helm upgrade django-app-<YourName> . --set replicaCount=3

This command terminates the current running pod and spins up three in its place. 

If we run the familiar command we've been using:

    $ kubectl get pods -namespace <env-var>

You'll see the pod replication in progress. 

### Tear it all down
Finally, we have built are app and deployed to the cluster. There is only one thing left to do. Destroy it. 

To delete the deployment you simply run:

    helm del --purge django-app-<YourName>

## Next steps
The next step will be to create your own Helm Chart. You can try this with an application of your own or run through [Bitnami's excellent guide](https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/) on how to build using a simple quickstart. 

