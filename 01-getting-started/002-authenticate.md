---
category: cloud-platform
expires: 2019-01-31
---
# Authenticating to the Cluster

## Aim

The aim of this document is to provide you with the necessary guidance to authenticate to a Cloud Platform cluster. When complete you should have access to perform API calls via a tool called `kubectl`.

## Installation

To begin, you must install the `kubectl` cli tool provided by Kubernetes. If you're using macOS then this is a simple case of running:

`$ brew install kubectl`

For other Operating Systems please see the following document: https://kubernetes.io/docs/tasks/tools/install-kubectl/

You must also have a valid GitHub account, be a member of the Ministry of Justice Organisation and belong to at least one team.

## Clusters

#### Introduction
At the time of writing this document the Cloud Platform team has two clusters:

[**Non-production cluster**](login.apps.non-production.k8s.integration.dsd.io)
[**Test cluster**](login.apps.cloud-platforms-test.k8s.integration.dsd.io)

Authentication to these clusters is handled via a third party authentication helper named [Kuberos](https://github.com/negz/kuberos). For our use case, we want authentication and identity to be handled by Github, and to derive all cluster access control from Github teams - projects will be deployed into namespaces (e.g. `pvb-production`, `cla-staging`), and access to resources in those namespaces should be available to the appropriate teams only (e.g. `PVB` and `CLA` teams).

Kubernetes supports authentication from external identity providers, including group definition, via [OIDC](https://kubernetes.io/docs/admin/authentication/#openid-connect-tokens). Github however only support OAuth2 as an authentication method, so an identity broker is required to translate from OAuth2 to OIDC.

As work on MOJ's identity service is ongoing, a development [Auth0](https://www.auth0.com) account has been created to act as a standin in the meantime.

#### How do I connect to a cluster?

To authenticate to any of the three clusters, please follow the steps below;

 - Pick the cluster you'd like to authenticate against:
[Non-production cluster](login.apps.non-production.k8s.integration.dsd.io)
[Test cluster](login.apps.cloud-platforms-test.k8s.integration.dsd.io)
 - Click the login with GitHub option and authorise Kuberos.
 - Follow the instructions on the page presented, once finished you should have a config file in the  directory as shown below.
 `~/.kube/config`

 - You should now be able to run `kubectl` commands such as `$ kubectl get pods --namespace <nameSpace>`

## Useful commands
```
$ kubectl config view # Show Merged kubeconfig settings.

# use multiple kubeconfig files at the same time and view merged config
$ KUBECONFIG=~/.kube/config:~/.kube/kubconfig2 kubectl config view

$ kubectl config current-context              # Display the current-context
$ kubectl config use-context my-cluster-name  # set the default context to my-cluster-name

# add a new cluster to your kubeconf that supports basic auth
$ kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# set a context utilizing a specific username and namespace.
$ kubectl config set-context gce --user=cluster-admin --namespace=foo \
  && kubectl config use-context gce
```
For more information, please see this handy [cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

 ## Where to go from here?
 Why don't you try and create a namespace of your own and deploying an application.
 [Create a namespace](https://ministryofjustice.github.io/cloud-platform-user-docs/cloud-platform/env-create/#creating-a-cloud-platform-environment)
 [Deploying an application](https://ministryofjustice.github.io/cloud-platform-user-docs/cloud-platform/app-deploy/#deploying-an-application-to-the-cloud-platform)
