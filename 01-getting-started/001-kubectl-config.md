---
category: Setup
expires: 2018-01-31
---
# `kubectl` configuration

## Overview

The aim of this guide is to provide a walkthrough of the installation of the `kubectl` command-line tool, the official command-line tool for Kubernetes, as well as setting it up by authenticating with the Cloud Platform cluster.

`kubectl` is used to deploy and manage applications on Kubernetes and is fundamental for anyone who wants to interact with the cluster.

When complete you should have access to perform API calls via a tool called `kubectl`.

## Installation

Please read the [official documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/) on how to install `kubectl`.

## Authentication

You must have a GitHub account and be a member of the Ministry of Justice Organisation.

For our use case, we want authentication and identity to be handled by Github, and to derive all cluster access control from Github teams - projects will be deployed into namespaces (e.g. `pvb-production`, `cla-staging`), and access to resources in those namespaces should be available to the appropriate teams only (e.g. `PVB` and `CLA` teams).

Kubernetes supports authentication from external identity providers, including group definition, via [OIDC](https://kubernetes.io/docs/admin/authentication/#openid-connect-tokens). Github however only support OAuth2 as an authentication method, so an identity broker is required to translate from OAuth2 to OIDC.

As work on MOJ's identity service is ongoing, a development [Auth0](https://www.auth0.com) account has been created to act as a standin in the meantime.

### Live clusters

Live clusters are those available to users:

| Cluster Name | Login page |
| ------------ | ---------- |
| `cloud-platform-live-0` | https://login.apps.cloud-platform-live-0.k8s.integration.dsd.io/ |

### How do I connect to a cluster?

We employ [Kuberos](https://github.com/negz/kuberos) to help with the setup, a service that can generate the client configuration for users.

To authenticate with a cluster, please follow the steps below;

 - Navigate to a login page from the table above
 - Click the login with GitHub option and authorise kuberos
 - Follow the instructions on the page presented, once finished you should have a [`kubeconfig`](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) file in `~/.kube/config`.
 - You should now be able to run `kubectl` commands; try running such `kubectl get namespaces`

### Multiple clusters
To setup additional clusters, follow the process above and save the generated `kubeconfig` with a different filename (eg.: `~/.kube/config_live0`).

You can then use the `KUBECONFIG` environment variable to have `kubectl` parse the additional configuration files, eg.: `KUBECONFIG=~/.kube/config:~/.kube/config_live0`

For more information please read the [official documentation](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

#### Usernames
By default, Kuberos will use your email address as the username in the generated `kubeconfig`.

When setting up multiple clusters, this will generate conflicts so you should rename the users by editing the files (and update the contexts appropriately).

## Where to go from here?

Now that you've setup `kubectl`, you might want to look at this handy [cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

Once you are ready to deploy applications you will need to create an environment first:

 * [Create a namespace]({{ "/01-getting-started/003-env-create" | relative_url }})
 * [Deploying an application]({{ "/02-deploying-an-app/001-app-deploy-helm" | relative_url }})
