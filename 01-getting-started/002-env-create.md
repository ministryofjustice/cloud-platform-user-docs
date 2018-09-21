---
category: cloud-platform
expires: 2019-01-31
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
$ git checkout -b <yourBranch>
```

### The directory structure

We build new environments by creating a new directory for our environment and putting the YAML files that define the environment into that directory. To understand where to create the directory it is useful to understand the overall structure of the repo:  

```
cloud-platform-environments
├── namespace-resources
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

Within the cluster directory you will create a directory for your environment in the format `<servicename-env>`, for example `myapp-dev`.

**/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/<servicename-env>**

The `<servicename-env>` directory for your environment defines the specific resources we will create in your namespace. In this guide we create the base namespace definition and a rolebinding that sets who can perform actions on the namespace.

## Define your environment

We create the environment by adding a directory in the `/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/` directory.

To set up an environment we need to create 5 files in our namespace:

* [`00-namespace.yaml`](#00-namespaceyaml)
* [`01-rbac.yaml`](#01-rbacyaml)
* [`02-limitrange.yaml`](#02-limitrangeyaml)
* [`03-resourcequota.yaml`](#03-resourcequotayaml)
* [`04-networkpolicy.yaml`](#04-networkpolicyyaml)

These files define key elements of the namespace and restrictions we want to place on it so that we have security and resource allocation properties. We describe each of these files in more detail below.

### `00-namespace.yaml`

In Kubernetes you run your application in a namespace; namespaces allow us to subdivide our cluster for multitenacy and provide isolation of Kubernetes resources.

### `01-rbac.yaml`

Role base access control will provide permission to grant access to resources within your namespace. You will need to provide your github team that matches the Github API as your github team will be given access to your namespace via this file.

### `02-limitrange.yaml`

As we are working on a shared Kubernetes cluster it is useful to put in place limits on the resources that each namespace, pod and container can use. This helps to stop us accidentally entering a situation where a one service impacts the performance of another through using more resources than are available.

### `03-resourcequota.yaml`

The ResourceQuota object allows us to set a total limit on the resources used in a namespace

### `04-networkpolicy.yaml`

The NetworkPolicy object defines how groups of pods are allowed to communicate with each other and other network endpoints. By default pods are non-isolated, they accept traffic from any source. We apply a network policy to restrict where traffic can come from, allowing traffic only from the ingress controller and other pods in your namespace.

## Create your namespace and namespace-resources

We will automate the creation of these namespace-resources files using terraform. You will need to install terraform locally.

```Shell
$ brew install terraform
```

In each of these files you need to replace some example values with information about your namespace, team or app. We do this by running terraform commands and filling in the values.

```Shell
$ terraform init
$ terraform plan
$ terraform apply
```
Our terraform module uses live-0 by default but if you would like to deploy to another cluster.

```Shell
$ terraform apply -var "cluster=<cluster-name>"
```

## Inputs

These are the inputs for the terraform module, that you will need to fill.

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| application |  | string | - | yes |
| business-unit | Area of the MOJ responsible for the service | string | - | yes |
| cluster | What cluster are you deploying your namespace. I.E cloud-platform-test-1 | string | `cloud-platform-live-0` | no |
| contact_email | Contact email address for owner of the application | string | - | yes |
| environment |  | string | - | yes |
| github_team | This is your team name as defined by the GITHUB api. This has to match the team name on the Github API | string | - | yes |
| is-production |  | string | `false` | no |
| namespace | Namespace you would like to create on cluster <application>-<environment>. I.E myapp-dev | string | - | yes |
| owner | Who is the owner/Who is responsible for this application | string | - | yes |
| source_code_url | Url of the source code for your application | string | - | yes |



You can access your namespace files under cloud-platform-environments/$namespaces/$cluster/$your-namespace, if satisfied you can then push the changes to your branch and create a pull request against the [`cloud-platform-environments`](https://github.com/ministryofjustice/cloud-platform-environments) master repo.

The cloud platform team will merge the pull request which will kick off the pipeline that builds the environment. You can check whether the build succeeded or failed in the [`#cp-build-notifications`](https://mojdt.slack.com/messages/CA5MDLM34/) slack channel.  


## Accessing your environments

You will need to commit these changes to github. Once your branch has been merged to master, our pipeline will create your namespace and deploy the resources to it.

Once the pipeline has completed you will be able to check that your environment is available by running:

`$ kubectl get namespaces`

This will return a list of the namespaces within the cluster, and you should see yours in the list.

You can now run commands in your namespace by appending the `-n` or `--namespace` flag to `kubectl`. So for instance, to see the pods running in our `myenv-dev` namespace, we would run:

`$ kubectl get pods -n myenv-dev` or

`$ kubectl get pods --namespace myenv-dev`

## Next steps
[Create an ECR repository]({{ "/01-getting-started/003-ecr-setup" | relative_url }}) to push your application docker image to.

Then you can try [deploying an app to Kubernetes manually]({{ "/02-deploying-an-app/001-app-deploy" | relative_url }}) by writing some custom YAML files or [deploying an app with Helm]({{ "/02-deploying-an-app/002-app-deploy-helm" | relative_url }}), a Kubernetes [package manager]((https://helm.sh/)).
