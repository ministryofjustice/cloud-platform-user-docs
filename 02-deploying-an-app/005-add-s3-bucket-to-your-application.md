---
category: cloud-platform
expires: 2018-01-31
---

# Adding S3 Bucket for your application.

## Introduction
This document will cover how to add an S3 bucket for your application using our [s3 bucket terraform module](https://github.com/ministryofjustice/cloud-platform-terraform-s3-bucket).


### Creating S3 Bucket

1\. In order to create a S3 bucket, you will need to work within our [cloud-platform-environments](https://github.com/ministryofjustice/cloud-platform-environments) repo. Git clone the repo and create a new branch.

```bash

  $ git clone git@github.com:ministryofjustice/cloud-platform-environments.git #git clone repo

  $ cd cloud-platform-environments # navigate into cloud-platform-environments directory.

  $ git co -b add_s3_resource   # create and checkout new branch.

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

4\. Create your module and change the module name and input variables. More instructions [here](https://github.com/ministryofjustice/cloud-platform-terraform-s3-bucket). Follow the example directory for more direction. Also create a Kubernetes secret, which will use the values from your S3 and create a Kubernetes Secret within our cluster. More information [here](https://www.terraform.io/docs/providers/kubernetes/r/secret.html).


```hcl

  $ cat main.tf
  terraform {
    backend "s3" {}
  }

  provider "aws" {
    region = "eu-west-1"
  }

  module "example_team_s3" {
    source = "github.com/ministryofjustice/cloud-platform-terraform-s3-bucket?ref=master"

    team_name              = "cloudplatform"
    bucket_identifier      = "example-bucket"
    acl                    = "public-read"
    versioning             = true
    business-unit          = "mojdigital"
    application            = "cloud-platform-terraform-s3-bucket"
    is-production          = "false"
    environment-name       = "development"
    infrastructure-support = "platform@digtal.justice.gov.uk"
  }

  resource "kubernetes_secret" "example_s3_bucket_credentials" {
    metadata {
      name      = "s3-bucket-example"
      namespace = "platforms-dev"
    }

    data {
      bucket_name       = "${module.example_team_s3.bucket_name}"
      access_key_id     = "${module.example_team_s3.access_key_id}"
      secret_access_key = "${module.example_team_s3.secret_access_key}"
    }
  }

```

5\. Once the terraform file created, git add, commit and push to your branch.

```bash

  $ git add main.tf

  $ git commit -m "created terraform module for s3 bucket"

  $ git push

```

6\. Make a pull request to merge your branch to master on the Github site.

7\. Once your request has been approved and the branches are merged, it will trigger our build pipeline which will apply the terraform module and create the resources.

8\. Here is an [working example](https://github.com/ministryofjustice/cloud-platform-environments/tree/add/terraform/namespaces/cloud-platform-test-1.k8s.integration.dsd.io/platforms-dev)

### Resources created with S3 Module.
The AWS resoures that are created as part of the S3 module.

- S3 Bucket
- IAM User (The IAM user will have access to the bucket and objects stored within the bucket)
- Credentials for IAM user (AWS_ACCESS_KEY and AWS_SECRET_KEY)

### Accessing the credentials

The end result will be a kubernetes `Secret` inside your namespace, called `s3-bucket-example`; the secret holds IAM credentials to authenticate and the bucket name.

To retrieve the credentials:
```
kubectl -n platforms-dev get secret s3-bucket-example -o yaml
```
