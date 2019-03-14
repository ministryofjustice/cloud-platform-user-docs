---
category: cloud-platform
expires: 2018-06-30
---
# Live 1 Migration Checklist

## Overview

After some long consideration of possible options, the decision has been made to migrate from the `live-0` cluster to the new `live-1` cluster.

The reason behind this decision is based on the need to move to a dedicated AWS account, which will be much easier to support, and the need to move away from the Ireland (EU) region to the London (UK) region.

The purpose of this document is to aid development teams in migrating their existing applications from `live-0` to `live-1`.

## Migration Steps

The migration steps that need to be taken may differ for individual applications. 

The following steps are for an application that is considered to be fairly normal.

Following these steps are a few extra consideration points, that are not covered in the example, but may apply to your application.

### Accessing the Live-1 cluster

To access the `live-1` cluster, navigate to the [Kuberos configuration page](https://login.apps.live-1.cloud-platform.service.justice.gov.uk), and download your Kube config file.

Kubernetes provides a [brief guide](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable) on how to set up `kubectl` to use multiple config files simultaneously.

You should now be able to switch contexts between the `live-0` and `live-1` clusters.

