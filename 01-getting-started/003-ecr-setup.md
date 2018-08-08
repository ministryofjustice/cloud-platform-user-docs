---
category: cloud-platform
expires: 2018-01-31
---
# Creating an ECR repository

## Introduction

This guide will guide you through the creation of an ECR (Elastic Container Registry) repository for your application's docker image.

AWS resources are provisioned through the [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments/) repository, per environment. Your application might be using multiple environments, however, you only need one image repository so we will be working in only one of the environments (it doesn't matter which one).

## Write the terraform configuration
In your local clone of the environments repository, `cd` to your environment's path and create a new directory called `resources`. Inside `resources`, create a new file called `ecr.tf`:

```
$ cat > ecr.tf
terraform {
  backend "s3" {}
}

provider "aws" {
  region = "eu-west-1"
}

module "ecr-repo" {
  source = "github.com/ministryofjustice/cloud-platform-terraform-ecr-credentials?ref=master"

  team_name = "my-team-name"
  repo_name = "my-app-name"
}

resource "kubernetes_secret" "ecr-repo" {
  metadata {
    name      = "ecr-repo-my-app-name"
    namespace = "my-environment"
  }

  data {
    repo_url          = "${module.ecr-repo.repo_url}"
    access_key_id     = "${module.ecr-repo.access_key_id}"
    secret_access_key = "${module.ecr-repo.secret_access_key}"
  }
}
```
Type [Ctrl-d] at the end to save the file.

This will create an image repository at `<account_number>.dkr.ecr.eu-west-1.amazonaws.com/my-team-name/my-app-name` along with the credentials to push to it.

Commit the changes and raise a pull request. Once merged, our pipeline will apply the changes and create the AWS resources.

For more information about the terraform module being used, please read the documentation [here](https://github.com/ministryofjustice/cloud-platform-terraform-ecr-credentials).

## Accessing the credentials

The end result will be a kubernetes `Secret` inside your environment, called `ecr-repo-my-app-name`; the secret holds IAM access keys to authenticate with the registry and the actual repository URL.

To retrieve the credentials:
```
kubectl -n <environment_name> get secret ecr-repo-my-app-name -o yaml
```

The values in kubernetes `Secrets` are always `base64` encoded so you will have to decode them before you can use them outside kubernetes. Inside the cluster, the nodes already have access to the ECR so you don't need to make any changes.

### Setting up CircleCI
In your CircleCI project, go to the settings (the cog icon) and select 'AWS Permissions' from the left hand menu. Fill in the IAM credentials and CircleCI will be able to use ECR images. For more information please see [the official docs](https://circleci.com/docs/2.0/private-images/).


## Next steps

Try [deploying an app]({{ "/02-deploying-an-app/001-app-deploy-helm" | relative_url }}) with [Helm](https://helm.sh/), a Kubernetes package manager, or [deploying manually]({{ "/02-deploying-an-app/002-app-deploy" | relative_url }}) by writing some custom YAML files.
