---
category: cloud-platform
expires: 2019-08-12
---

# Zero Downtime Deployments

## Introduction

In the past at the MoJ, application team members were used to application service outage when deploying a new version of their application.

With the Cloud Platform, this is now an annoyance of the past, thanks to Rolling Updates.

## Rolling Updates

Rolling updates introduce the ability to update an application without any downtime.

A rolling update works by ensuring there is always one extra Pod than the maximum number stated in the deployment.

The new deployment is applied incrementally, one Pod at a time until all Pods are running the latest version.

If an application is exposed publicly, traffic will only be routed to the available Pods.

### The Major Advantage

Where Rolling Updates really brings benefit is the enabling of Continuous Integration and Continuous Delivery.

The ability to constantly update your application with micro-changes, with zero downtime, brings a host of benefits.

### Further Reading

If you'd like to read more in-depth into Rolling Updates, then Kubernetes has some great documentation [here](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/).

