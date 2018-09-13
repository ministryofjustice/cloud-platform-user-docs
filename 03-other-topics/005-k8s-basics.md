---
category: cloud-platform
expires: 2019-01-01
---
# Kubernetes Basics - Links to helpful beginner resources

The purpose of this page is to serve as a small catalogue of links to basic Kubernetes concepts and learning resources.

## The Concept

Kubernetes is very often referred to as **"K8s"**, purely for the reason, that typing Kubernetes over and over gets quite taxing.

If Kubernetes is a totally new concept to you, then firstly you should probably check out a few of the resources linked below:

Official Website: 
https://kubernetes.io/

Offical Docs: 
https://kubernetes.io/docs/concepts/

Quick concept video: 
https://www.youtube.com/watch?v=IMOZCDhH7do

## Usage Platforms

Kubernetes is normally used with one of the big 3 cloud providers. Amazon Web Services, Google Cloud Engine, and Microsoft Azure.

All 3 have their own documentation and implementation / hosting tools:

### AWS

Kubernetes Docs: 
https://kubernetes.io/docs/setup/turnkey/aws/

Implementation Tool:
https://aws.amazon.com/eks/ 

### GCE

Kubernetes Docs: 
https://kubernetes.io/docs/setup/turnkey/gce/

Implementation Tool:
https://cloud.google.com/kubernetes-engine/

### Local Hosting

To keep costs low and for learning the basics, it is possible to host a Kubernetes Cluster locally on your machine. Docker and Minikube both provide tools for this:

Docker: https://docs.docker.com/docker-for-mac/kubernetes/

Minikube: https://kubernetes.io/docs/setup/minikube/


### Azure

Kubernetes Docs: 
https://kubernetes.io/docs/setup/turnkey/azure/

Implementation Tool:
https://azure.microsoft.com/en-gb/services/kubernetes-service/

## Command Line Kubernetes Interaction

Kubernetes has it's own official CLI tool for interacting with a Cluster called `kubectl`. It is certainly worth learning the basics of `kubectl`, As the vast majority of time interacting with Kubernetes will be through this tool.

Overview: https://kubernetes.io/docs/reference/kubectl/overview/

Installation: https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Online Courses

You will find a multitude of courses online that offer to teach the basics of Kubernetes (and beyond). Some are better than others, some are free, and some require a subscription or one-off payment for access.

The two courses we recommend are:

### Pluralsight
https://www.pluralsight.com/courses/getting-started-kubernetes

### Udacity
https://eu.udacity.com/course/scalable-microservices-with-kubernetes--ud615