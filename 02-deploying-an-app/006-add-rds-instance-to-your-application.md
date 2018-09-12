# Adding RDS Instance to your application

## Introduction
This document will cover how to add an RDS instance to your application using our [rds instance terraform module](https://github.com/ministryofjustice/cloud-platform-terraform-rds-instance).

### Creating an RDS Instance

1\. In order to create a RDS Instance, you will need to work within our [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments) repo. Git clone the repo and create a new branch.

```bash

  $ git clone git@github.com:ministryofjustice/cloud-platform-environments.git #git clone repo

  $ cd cloud-platform-environments # navigate into cloud-platform-environments directory.

  $ git co -b add_rds_instance   # create and checkout new branch.

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

4\. Create your module and change the module name and input variables. More instructions [here](https://github.com/ministryofjustice/cloud-platform-terraform-rds-instance). Follow the example directory for more direction. Also create a Kubernetes secret, which will use the values from your S3 and create a Kubernetes Secret within our cluster. More information [here](https://www.terraform.io/docs/providers/kubernetes/r/secret.html).


```hcl

  $ cat main.tf
  terraform {
    backend "s3" {}
  }

  provider "aws" {
    region = "eu-west-1"
  }

  // The following two variables are provided at runtime by the pipeline.
  variable "cluster_name" {}
  variable "cluster_state_bucket" {}

  module "example_team_rds" {
    source = "github.com/ministryofjustice/cloud-platform-terraform-rds-instance?ref=2.0"

    cluster_name               = "${var.cluster_name}"
    cluster_state_bucket       = "${var.cluster_state_bucket}"
    team_name                  = "example-repo"
    db_allocated_storage       = "100"
    db_engine                  = "postgres"
    db_engine_version          = "10.4"
    db_instance_class          = "db.t2.small"
    db_backup_retention_period = 10
    db_storage_type            = "io1"
    db_iops                    = "1000"
    business-unit              = "example-bu"
    application                = "exampleapp"
    is-production              = "false"
    environment-name           = "development"
    infrastructure-support     = "example-team@digtal.justice.gov.uk"
  }

  resource "kubernetes_secret" "example_rds_instance_credentials" {
    metadata {
      name      = "rds-instance-example"
      namespace = "platforms-dev"
    }

    data {
      endpoint          = "${module.example_team_rds.rds_instance_endpoint}"
      database_name     = "${module.example_team_rds.database_name}"
      database_username = "${module.example_team_rds.database_username}"
      database_password = "${module.example_team_rds.database_password}"
        }
    }
  }

```

5\. Once the terraform file created, git add, commit and push to your branch.

```bash

  $ git add main.tf

  $ git commit -m "created terraform module for rds instance"

  $ git push

```

6\. Make a pull request to merge your branch to master on the Github site.

7\. Once your request has been approved and the branches are merged, it will trigger our build pipeline which will apply the terraform module and create the resources.

<!-- 8\. Here is an [working example](https://github.com/ministryofjustice/cloud-platform-environments/tree/add/terraform/namespaces/cloud-platform-test-1.k8s.integration.dsd.io/platforms-dev) -->

### Resources created with RDS Instance
The AWS resoures that are created as part of the RDS instance.

- RDS Instance
- RDS Database Subnet Group
- KMS key
- IAM User (The IAM user will have access to the RDS instance.
- Credentials for IAM user (AWS_ACCESS_KEY and AWS_SECRET_KEY)

### Accessing the credentials

The end result will be a kubernetes `Secret` inside your namespace, called `rds-instance-example`; the secret holds username and IAM credentials for the database and its address.

To retrieve the credentials:
```
kubectl -n platforms-dev get secret rds-instance-example -o yaml
```
