# Adding an DynamoDB cluster to your application

## Introduction
This document will cover how to add DynamoDB to your application using our [terraform module](https://github.com/ministryofjustice/cloud-platform-terraform-dynamodb-cluster).

### Creating a cluster

sddsf

1\. In order to create an DynamoDB cluster, you will need to work within our [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments) repo. Git clone the repo and create a new branch.

```bash

  $ git clone git@github.com:ministryofjustice/cloud-platform-environments.git #git clone repo
namespaces
asa
  $ cd cloud-platform-environments # navigate into cloud-platform-environments directory.

  $ git co -b add_dynamodb   # create and checkout new branch.

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

4\. Create your module and change the module name and input variables. More instructions [here](https://github.com/ministryofjustice/cloud-platform-terraform-dynamodb-cluster). Follow the example directory for more direction. Also create a Kubernetes secret, which will use the values from the Terraform output and add it to your namespace. More information [here](https://www.terraform.io/docs/providers/kubernetes/r/secret.html).


```hcl

terraform {
  backend "s3" {}
}

provider "aws" {
  region  = "eu-west-1"
  version = "~> 1.17.0"
}

module "example_team_dynamodb" {
  source = "github.com/ministryofjustice/cloud-platform-terraform-dynamodb-cluster?ref=v1.0"

  team_name              = "example-team"
  business-unit          = "example-bu"
  application            = "exampleapp" # note RDS doesn't accept "-" in the DB name
  is-production          = "false"
  environment-name       = "development"
  infrastructure-support = "example-team@digtal.justice.gov.uk"

  hash_key  = "example-hash"
  range_key = "example-range"
}

resource "kubernetes_secret" "dynamodb" {
  metadata {
    name      = "dynamodb"
    namespace = "exampleapp"
  }

  data {
    name = "${module.dynamodb.table_name}"
    access_key_id     = "${module.dynamodb.access_key_id}"
    secret_access_key = "${module.dynamodb.secret_access_key}"
  }
}

```

5\. Once the terraform file created, git add, commit and push to your branch.

```bash

  $ git add main.tf

  $ git commit -m "created Terraform module for DynamoDB"

  $ git push

```

6\. Make a pull request to merge your branch to master on the Github site.

7\. Once your request has been approved and the branches are merged, it will trigger our build pipeline which will apply the terraform module and create the resources.

### Resources created for DynamoDB
AWS resources that are created as part of the pipeline:

 - a "simple" (as opposed to "global") table
 - an autoscaling role&policy, CloudWatch based
 - a dedicated IAM user and API key allowing only access to this resource
