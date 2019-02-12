---
category: cloud-platform
expires: 2019-08-12
---

# Zero Downtime Deployments

## Introduction

Zero Downtime Deployments are a significant feature of the Cloud Platform, that bring a host of advantages.

## Rolling Updates

Rolling updates introduce the ability to update an application without any downtime.

A rolling update works by ensuring there is always one extra Pod than the maximum number stated in the deployment.

The new deployment is applied incrementally, usually one to two Pods at a time until all Pods are running the latest version. How the deployment is rolled out, can be configured by the user. More information about configuring a rolling update can be found [here](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/).

If an application is exposed publicly, traffic will only be routed to the available Pods. However, this is only the case if the user configured `readinessProbes` correctly. This is a large topic, so more information can be found [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).

### The Major Advantage

Where rolling updates really brings benefit is the enabling of Continuous Integration and Continuous Delivery.

The ability to constantly update your application, with zero downtime, brings a host of benefits.

### Further Reading

If you'd like to read more in-depth into rolling updates, then Kubernetes has some great documentation [here](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/).



