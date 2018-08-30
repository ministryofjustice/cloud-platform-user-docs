---
category: cloud-platform
expires: 2018-01-31
---
# Creating a Cloud Platform Environment

## Introduction

This is a guide to creating a environment in one of our Kubernetes clusters.

We define an environment as a Kubernetes [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) with some key resources deployed in it. Each Kubernetes namespace creates a logical separation within our cluster that provides isolation from any other namespace.

Once you have created an environment you will be able to perform actions using the `kubectl` tool in the namespace you have created.

## Objective

By the end of this guide you'll have created a Kubernetes namespace ready for you to [deploy an application]({{ "/02-deploying-an-app/001-app-deploy-helm" | relative_url }}) into.

## Create an environment

You create an environment by adding the definition of the environment in YAML to the following repository, hosted on GitHub:

[cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments)

Adding your environment definition kicks off a pipeline which builds your environment on the appropriate cluster.  

### Set up

First we need to clone the repository, change directory and create a new branch:

```
$ git clone git@github.com:ministryofjustice/cloud-platform-environments.git
$ cd cloud-platform-environments
$ git checkout -b "yourBranch"
```

### The directory structure

We build new environments by creating a new directory for our environment and putting the YAML files that define the environment into that directory. To understand where to create the directory it is useful to understand the overall structure of the repo:  

```
cloud-platform-environments
└── namespaces
    └── cloud-platform-live-0.k8s.integration.dsd.io
        ├── kube-system

        ...

        └── user-roles.yaml
```

**cloud-platform-environments**

This is the root of the repo, containing `namespaces` directory

**/namespaces**

The namespaces directory contains a directory for each of the clusters that you can build environments on. Create your environment in the `cloud-platform-live-0.k8s.integration.dsd.io` directory.

**/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/**

Within the cluster directory you will create a directory for your environment in the format `$servicename-$env`, for example `myapp-dev`.

**/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/$servicename-$env**

The `$servicename-$env` directory for your environment defines the specific resources we will create in your namespace. In this guide we create the base namespace definition and a rolebinding that sets who can perform actions on the namespace.

### Defining your environment

We create the environment by adding a directory in the `/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/` directory:

```
$ cd namespaces/cloud-platform-live-0.k8s.integration.dsd.io/
$ mkdir myapp-dev
```

Now we will need to create two files within our `$servicename-$env` directory,  `namespaces.yaml` and `$servicename-$env-admin-role.yaml`. The structures of both files are shown below.

#### Namespace definition

The `namespace.yaml` file defines the namespace so that the cluster Kubernetes knows to create a namespace and what to call it.

```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-dev # This is where you will define your $servicename-$env
  labels:
    name: myapp-dev # Also your $servicename-$env
```
Example from the cloud-platform-reference-app [namespace file](https://github.com/ministryofjustice/cloud-platform-environments/blob/master/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/reference-app/00-namespace.yaml).

#### Rolebinding

We will also create a `RoleBinding` resource by adding a `$servicename-$env-admin-role.yaml` file. This will provide us with [access policies](https://kubernetes.io/docs/admin/authorization/rbac/) on the namespace we have created in the cluster.

A role binding resource grants the permissions defined in a role to a user or set of users. A role can be another resource we can create but in this instance we will reference a Kubernetes default role `ClusterRole - admin`.

This `RoleBinding` resource references the `ClusterRole - admin` to provide  admin permissions on the namespace to the set of users defined under `subjects`. In this case, the `$yourTeam` GitHub group will have admin access to any resources within the namespace `myapp-dev`.

```YAML
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: myapp-dev-admins  # Your namespace with `-admin` e.g. `$servicename-$env-admin`
  namespace: myapp-dev # Your namespace `$servicename-$env`
subjects:
  - kind: Group
    name: "github:$yourTeam" # Make this the name of the GitHub team you want to give access to
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

After both files are created commit them and create a pull request against the [`cloud-platform-environments`](https://github.com/ministryofjustice/cloud-platform-environments) master repo.

The cloud platform team will merge the pull request which will kick off the pipeline that builds the environment. You can check whether the build succeeded or failed in the [`#cp-build-notifications`](https://mojdt.slack.com/messages/CA5MDLM34/) slack channel.  

### Accessing your environments

Once the pipeline has completed you will be able to check that your environment is available by running:

`$ kubectl get namespaces`

This will return a list of the namespaces within the cluster, and you should see yours in the list.

## Next steps
Perhaps you would like to [create an ECR repository]({{ "/01-getting-started/003-ecr-setup" | relative_url }}) to push your docker image to.

If you don't need one, you can try [deploying an app]({{ "/02-deploying-an-app/001-app-deploy-helm" | relative_url }}) with [Helm](https://helm.sh/), a Kubernetes package manager, or [deploying manually]({{ "/02-deploying-an-app/002-app-deploy" | relative_url }}) by writing some custom YAML files.
