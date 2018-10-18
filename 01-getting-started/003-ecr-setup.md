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

  $ cd namespaces/$your_service  #navigate to your service's directory.

  $ mkdir resources # make directory called resources

  $ cd namespaces/$your_service/resources

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

This will create an image repository at `<account_number>.dkr.ecr.eu-west-1.amazonaws.com/my-team-name/my-app-name` along with the credentials to push to it. Make sure you adjust the values of `team_name`, `repo_name` and `namespace` to what is appropriate for your environment.

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

This convenience function placed in `~/.bash_profile` will log you on all the team's repositories:
```
function ecr_login() {
 # kubectl context as defined in ~/.kube/config
 local ecr_ctx="live-cluster"
 # GitHub team name goes here
 local ecr_t="my-team-name"
 export AWS_DEFAULT_REGION="eu-west-1"
 local ecr_ns=$(kubectl --context ${ecr_ctx} -o json get ns | jq -r ".items[] | select(.metadata.name|test(\"${1}\")) | .metadata.name")
 local ecr_n
 for ecr_n in ${ecr_ns} ; do
  local ecr_rb=$(kubectl --context ${ecr_ctx} -n ${ecr_n} get rolebindings -o json 2>/dev/null | jq -r ".items[].subjects[].name | select(.==\"github:${ecr_t}\")")
  if [ "${ecr_rb}" == "github:${ecr_t}" ] ; then
   local ecr_id ecr_sec ecr_url ecr_name
   while read ecr_id ; do
    read ecr_sec
    read ecr_url
    ecr_id=$(echo -n ${ecr_id} | base64 -D)
    ecr_sec=$(echo -n ${ecr_sec} | base64 -D)
    ecr_url=$(echo -n ${ecr_url} | base64 -D)
    ecr_name=$(echo -n ${ecr_url} | cut -d'/' -f2,3)
    echo "logging on ${ecr_url}"
    eval $(AWS_ACCESS_KEY_ID=${ecr_id} AWS_SECRET_ACCESS_KEY=${ecr_sec} aws ecr get-login --no-include-email) 2>&1 | grep -v WARN
    AWS_ACCESS_KEY_ID=${ecr_id} AWS_SECRET_ACCESS_KEY=${ecr_sec} aws ecr describe-images --repository-name ${ecr_name} | jq -r '.imageDetails? | map(.imageTags[]?) | join (" ")'
   done < <(kubectl --context ${ecr_ctx} -n ${ecr_n} get secrets -o json | jq -r '.items[] | select(.data.repo_url) | .data.access_key_id,.data.secret_access_key,.data.repo_url')
  fi
 done
}
```

To use it, just call with a parameter matching one or more projects, you will be logged on for `docker pull/push` and also get a list of exisiting tags:
```
$ ecr_login laa
logging on dkr.ecr.eu-west-1.amazonaws.com/laa/backend
Login Succeeded
master.9a126c8 circleci-badge cloud-platform-ecr-repo ...
logging on dkr.ecr.eu-west-1.amazonaws.com/laa/frontend
Login Succeeded
circleci circleci.29b3ecd ...
```

### Setting up CircleCI
In your CircleCI project, go to the settings (the cog icon) and select 'AWS Permissions' from the left hand menu. Fill in the IAM credentials and CircleCI will be able to use ECR images. For more information please see [the official docs](https://circleci.com/docs/2.0/private-images/).


## Next steps

Try [deploying an app]({{ "/02-deploying-an-app/002-app-deploy-helm" | relative_url }}) with [Helm](https://helm.sh/), a Kubernetes package manager, or [deploying manually]({{ "/02-deploying-an-app/001-app-deploy" | relative_url }}) by writing some custom YAML files.
