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

We create the environment by adding a directory in the `/namespaces/cloud-platform-live-0.k8s.integration.dsd.io/` directory:

```
$ cd namespaces/cloud-platform-live-0.k8s.integration.dsd.io/
$ mkdir myapp-dev
```

To set up an environment we need to create 5 files in our namespace:

* [`00-namespace.yaml`](#00-namespaceyaml)
* [`01-rbac.yaml`](#01-rbacyaml)
* [`02-limitrange.yaml`](#02-limitrangeyaml)
* [`03-resourcequota.yaml`](#03-resourcequotayaml)
* [`04-networkpolicy.yaml`](#04-networkpolicyyaml)

These files define key elements of the namespace and restrictions we want to place on it so that we have security and resource allocation properties. We describe each of these files in more detail below.

To get started copy the example versions of these files from the [`namespace-resources`](https://github.com/ministryofjustice/cloud-platform-environments/tree/master/namespace-resources) directory in the `cloud-platform-evironments` repo into your `<servicename-env>` directory.

```Shell
# From the root of the cloud-platform-environments directory
$ cp namespace-resources/* namespaces/cloud-platform-live-0.k8s.integration.dsd.io/myapp-dev/
```

In each of these files you need to replace some example values with information about your namespace, team or app. We go through each of the changes below.

After you have changed the files commit them and create a pull request against the [`cloud-platform-environments`](https://github.com/ministryofjustice/cloud-platform-environments) master repo.

The cloud platform team will merge the pull request which will kick off the pipeline that builds the environment. You can check whether the build succeeded or failed in the [`#cp-build-notifications`](https://mojdt.slack.com/messages/CA5MDLM34/) slack channel.  

### `00-namespace.yaml`

The `00-namespace.yaml` file defines the namespace so that the cluster Kubernetes knows to create a namespace and what to call it.

```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-dev # This is where you will define your <servicename-env>
  labels:
    name: myapp-dev # Also your <servicename-env>
```

### `01-rbac.yaml`

We will also create a `RoleBinding` resource by adding the `01-rbac.yaml` file. This will provide us with [access policies](https://kubernetes.io/docs/admin/authorization/rbac/) on the namespace we have created in the cluster.

A role binding resource grants the permissions defined in a role to a user or set of users. A role can be another resource we can create but in this instance we will reference a Kubernetes default role `ClusterRole - admin`.

This `RoleBinding` resource references the `ClusterRole - admin` to provide  admin permissions on the namespace to the set of users defined under `subjects`. In this case, the `<yourTeam>` GitHub group will have admin access to any resources within the namespace `myapp-dev`.

```YAML
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: myapp-dev-admins  # Your namespace with `-admin` e.g. `<servicename-env>-admin`
  namespace: myapp-dev # Your namespace `<servicename-env>`
subjects:
  - kind: Group
    name: "github:<yourTeam>" # Make this the name of the GitHub team you want to give access to
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

### `02-limitrange.yaml`

As we are working on a shared Kubernetes cluster it is useful to put in place limits on the resources that each namespace, pod and container can use. This helps to stop us accidentally entering a situation where a one service impacts the performance of another through using more resources than are available.  

The first Kubernetes limit we can use is a [LimitRange](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/) which we define in `02-limitrange.yaml`.

The LimitRange object specifies two key resource limits on containers, `defaultRequest` and `default`. `defaultRequest` is the memory and cpu a container will request on startup. This is what the Kubernetes scheduler uses to determine whether there is enough space on the cluster to run your application and what you application will start up with when it is created. `default` is the limit at which your application will be killed or throttled.

In `02-limitrange.yaml` you need to change the value of the `namespace` key to match the name of your namespace in the form `<service-env>`. We have set default values for the limits in the templates. As you learn more about the behaviour of your applications on Kubernetes you can change them.  

```YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: limitrange
  namespace: myapp-dev # Your namespace `<servicename-env>`
spec:
  limits:
  - default:
      cpu: 1000m
      memory: 2Gi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

### `03-resourcequota.yaml`

The [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/) object allows us to set a total limit on the resources used in a namespace. As with the LimitRange, the `requests.cpu` and `requests.memory` limits set how much namespace will request on creation. The `limits.cpu` and `limits.memory` define the overall hard limits for the namespace.

In `03-resourcequota.yaml` you need to change the value of the `namespace` key to match the name of your namespace in the form `<service-env>`. We have set default values for the limits in the templates. As you learn more about the behaviour of your applications on Kubernetes you can change them.  

```YAML
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: myapp-dev # Your namespace `<servicename-env>`
spec:
  hard:
    requests.cpu: 4000m
    requests.memory: 8Gi
    limits.cpu: 6000m
    limits.memory: 12Gi
```

### `04-networkpolicy.yaml`

The [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) object defines how groups of pods are allowed to communicate with each other and other network endpoints. By default pods are non-isolated, they accept traffic from any source. We apply a network policy to restrict where traffic can come from, allowing traffic only from the [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress/) and other pods in your namespace.

In `04-networkpolicy.yaml` you need to change the value of the `namespace` key to match the name of your namespace in the form `<service-env>`.

```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default
  namespace: myapp-dev # Your namespace `<servicename-env>`
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-ingress-controllers
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          component: ingress-controllers
```

## Accessing your environments

Once the pipeline has completed you will be able to check that your environment is available by running:

`$ kubectl get namespaces`

This will return a list of the namespaces within the cluster, and you should see yours in the list.

You can now run commands in your namespace by appending the `-n` or `--namespace` flag to `kubectl`. So for instance, to see the pods running in our `myenv-dev` namespace, we would run:

`$ kubectl get pods -n myenv-dev` or

`$ kubectl get pods --namespace myenv-dev`

## Next steps
[Create an ECR repository]({{ "/01-getting-started/003-ecr-setup" | relative_url }}) to push your application docker image to.

Then you can try [deploying an app to Kubernetes manually]({{ "/02-deploying-an-app/001-app-deploy" | relative_url }}) by writing some custom YAML files or [deploying an app with Helm]({{ "/02-deploying-an-app/002-app-deploy-helm" | relative_url }}), a Kubernetes [package manager]((https://helm.sh/)).
