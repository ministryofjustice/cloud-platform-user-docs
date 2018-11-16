---
category: cloud-platform
expires: 2019-01-31
---
# Creating an ECR repository

## Introduction

This guide will guide you through the creation of an ECR (Elastic Container Registry) repository for your application's docker image.

AWS resources are provisioned through the [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments/) repository, per environment. Your application might be using multiple namespaces, however you typically only need one image repository and once created in any of them you can copy credentials for it to the others via `kubectl get/create secret`.

### Creating the registry

1\. In order to create the ECR Docker registry, git clone the repo and create a new branch.

```bash

  $ git clone git@github.com:ministryofjustice/cloud-platform-environments.git #git clone repo

  $ cd cloud-platform-environments # navigate into cloud-platform-environments directory.

  $ git checkout -b add_ecr   # create and checkout new branch.

```

2\. You will need to navigate to your service's directory which is located in the namespaces directory. Create a directory named "resources".

```bash

  $ cd namespaces/cloud-platform-live-0.k8s.integration.dsd.io/$your_service  #navigate to your service's directory.

  $ mkdir resources # make directory called resources

  $ cd namespaces/$your_service/cloud-platform-live-0.k8s.integration.dsd.io/resources

```

3\. Create a terraform file within the resources directory.

```bash

  $ vi main.tf  #create a terraform file.

```

4\. Add the following contents to the .tf file:

```
terraform {
  backend "s3" {}
}

provider "aws" {
  region = "eu-west-1"
}

module "ecr-repo" {
  source = "github.com/ministryofjustice/cloud-platform-terraform-ecr-credentials?ref=1.0"

  team_name = "<my-team>"
  repo_name = "<my-app>"
}

resource "kubernetes_secret" "ecr-repo" {
  metadata {
    name      = "<ecr-repo-my-app>"
    namespace = "<my-app-dev>"
  }

  data {
    repo_url          = "${module.ecr-repo.repo_url}"
    access_key_id     = "${module.ecr-repo.access_key_id}"
    secret_access_key = "${module.ecr-repo.secret_access_key}"
  }
}
```

This will create an image repository at `<account_number>.dkr.ecr.eu-west-1.amazonaws.com/my-team-name/my-app-name` along with the credentials to push to it. Make sure you adjust the values of `team_name`, `repo_name` `name` and `namespace` to what is appropriate for your environment.

5\. git add, commit and push to your branch.

```bash

  $ git add main.tf

  $ git commit

  $ git push

```
6\. Once your request has been approved and the branches are merged, it will trigger our build pipeline which will apply the Terraform module and create the resources.

For more information about the terraform module being used, please read the documentation [here](https://github.com/ministryofjustice/cloud-platform-terraform-ecr-credentials).

## Accessing the credentials

The end result will be a kubernetes `Secret` inside your environment, called `ecr-repo-my-app-name`; the secret holds IAM access keys to authenticate with the registry and the actual repository URL.

To retrieve the credentials:
```
kubectl -n <namespace_name> get secret ecr-repo-my-app-name -o yaml
```

The values in kubernetes `Secrets` are always `base64` encoded so you will have to decode them before you can use them outside kubernetes. Inside the cluster, the nodes already have access to the ECR so you don't need to make any changes.

### Setting up CircleCI
In your CircleCI project, go to the settings (the cog icon) and select 'AWS Permissions' from the left hand menu. Fill in the IAM credentials and CircleCI will be able to use ECR images. For more information please see [the official docs](https://circleci.com/docs/2.0/private-images/).


## Next steps

Try [deploying an app]({{ "/02-deploying-an-app/002-app-deploy-helm" | relative_url }}) with [Helm](https://helm.sh/), a Kubernetes package manager, or [deploying manually]({{ "/02-deploying-an-app/001-app-deploy" | relative_url }}) by writing some custom YAML files.
