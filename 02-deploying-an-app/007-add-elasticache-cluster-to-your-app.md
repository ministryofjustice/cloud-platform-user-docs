# Adding an ElastiCache cluster to your application

## Introduction
This document will cover how to add ElastiCache to your application using our [terraform module](https://github.com/ministryofjustice/cloud-platform-terraform-elasticache-cluster).

### Creating a cluster

1\. In order to create an ElastiCache cluster, you will need to work within our [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments) repo. Git clone the repo and create a new branch.

```bash

  $ git clone git@github.com:ministryofjustice/cloud-platform-environments.git #git clone repo

  $ cd cloud-platform-environments # navigate into cloud-platform-environments directory.

  $ git co -b add_elasticache   # create and checkout new branch.

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

4\. Create your module and change the module name and input variables. More instructions [here](https://github.com/ministryofjustice/cloud-platform-terraform-elasticache-cluster). Follow the example directory for more direction. Also create a Kubernetes secret, which will use the values from the Terraform output and add it to your namespace. More information [here](https://www.terraform.io/docs/providers/kubernetes/r/secret.html).


```hcl

terraform {
  backend "s3" {}
}

provider "aws" {
region = "eu-west-1"
}

module "example_team_ec_cluster" {
source = "github.com/ministryofjustice/cloud-platform-terraform-elasticache-cluster?ref=v1.0"

team_name                 = "example-repo"
ec_engine                 = "redis"
engine_version            = "4.0.10"
parameter_group_name      = "default.redis4.0"
node_type                 = "cache.m3.medium"
number_of_nodes           = 1
port                      = 6379
ec_subnet_groups          = ["subnet-a", "subnet-b", "subnet-c"]
business-unit             = "example-bu"
application               = "exampleapp"
is-production             = "false"
environment-name          = "development"
infrastructure-support    = "example-team@digtal.justice.gov.uk"
}

resource "kubernetes_secret" "elasticache" {
  metadata {
    name      = "elasticache"
    namespace = "exampleapp"
  }

  data {
    name = "${module.elasticache.endpoint}"
  }
}

```

5\. Once the terraform file created, git add, commit and push to your branch.

```bash

  $ git add main.tf

  $ git commit -m "created Terraform module for ElastiCache"

  $ git push

```

6\. Make a pull request to merge your branch to master on the Github site.

7\. Once your request has been approved and the branches are merged, it will trigger our build pipeline which will apply the terraform module and create the resources.

### Resources created for ElastiCache
AWS resources that are created as part of the pipeline:

 - a subnet group
 - 1 or more EC nodes
 - tags including the team name with a random ID appended
