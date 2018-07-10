---
category: cloud-platform
expires: 2018-01-31
---

# Adding S3 Bucket for your application.

## Introduction
This document will cover how to add an S3 bucket for your application using our terraform module. 


### Creating S3 Bucket
[Terraform S3 Bucket Repo](https://github.com/ministryofjustice/cloud-platform-terraform-s3-bucket)

[Cloud-Platform-Environments Repo](https://github.com/ministryofjustice/cloud-platform-environments)

1\. In order to create a S3 bucket, you will need to work within our Cloud-Platform-Environments repo. Git clone the repo and create a new branch.

    In terminal

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

4\. Create your module by changing the module name and variables. More instructions [here](https://github.com/ministryofjustice/cloud-platform-terraform-s3-bucket). Follow the example directory for more direction.


```hcl
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
