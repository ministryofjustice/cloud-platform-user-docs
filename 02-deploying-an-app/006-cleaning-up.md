# Cleaning up

When you have finished working with your initial deployment, please clean up the resources you have created. This helps to keep the cloud platform repositories well-organised, and speeds up deployments and changes to the cluster (because the build process doesn't have to spend time managing unnecessary resources). It also helps to keep our hosting costs down.

The resources to be removed are:

* The ECR which stores your docker images
* Your namespace in the cluster. This contains all of the pods, containers and other cluster resources for your application. Removing the cluster namespace will automatically clean up all of its contents.

## Removing your ECR

Your [ECR][ecr] was created by adding an `ecr.tf` file to the Cloud Platform [environments repository][envrepo].

To delete your ECR, once you no longer need it, requires two steps:

1. Remove the `ecr.tf` file from the [environments repository][envrepo]. This prevents the ECR from being automatically recreated the next time the cluster configuration is applied.
1. A cluster administrator needs to manually delete the ECR (either via the Amazon AWS web console, or using the [AWS CLI][awscli].

## Removing your namespace

Namespaces are created by adding YAML config files to the Cloud Platform [environments repository][envrepo].

To delete a namespace requires two steps:

1. Remove the YAML config files from the [environments repository][envrepo]. This prevents the namespace from being automatically recreated the next time the cluster configuration is applied.
1. A cluster administrator needs to run a `kubectl delete namespace [name of the namespace]` command to delete the namespace and its contents.

## Raising a pull request

When you have finished with your sandbox namespace, and its associated ECR, please create a fork of the [environments repository][envrepo] with your environment's sub-directory of the `namespaces/cloud-platform-live-0.k8s.integration.dsd.io` directory removed, and raise a [pull request][pr] to merge your fork into master. Once merged, this will remove the YAML files which define your namespace, and also the `ecr.tf` file which defines your ECR.

In the body of your PR, please add a note to ask the cloud platform team to manually delete both your namespace and your ECR.

[envrepo]: https://github.com/ministryofjustice/cloud-platform-environments
[ecr]: https://aws.amazon.com/ecr/
[awscli]: https://aws.amazon.com/cli/
[pr]: https://help.github.com/en/articles/about-pull-requests
