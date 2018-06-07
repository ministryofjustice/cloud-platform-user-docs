---
category: cloud-platform
expires: 2018-01-31
---

# Continuous Deployment of an application using CircleCI and Helm

## Introduction
This document covers how to continuously deploy your application in the Cloud Platform. It is essentially a continuation of [‘Deploying an application to the Cloud Platform with Helm’](https://ministryofjustice.github.io/cloud-platform-user-docs/02-deploying-an-app/001-app-deploy-helm/#tutorial-deploying-an-application-to-the-cloud-platform-with-helm).

*Note: This document is specific to using [CircleCI](https://circleci.com/) as the Continuous Integration method.*

### Objective
By the end of the tutorial, you will have done the following:

- Created a Service Account for CircleCI in your namespace
- Generated a CircleCI configuration file in your application repository. The configuration file will authenticate with your chosen cluster, build a docker image and push it to ECR and upgrade your helm deployment with the new docker image.
- Have an automated CircleCI pipeline that upgrades your helm deployment when a new change is pushed to your master branch

### Requirements
It is assumed you have the following:

 - You have [created an environment for your application](/01-getting-started/003-env-create)
 - You have [deployed an application](https://ministryofjustice.github.io/cloud-platform-user-docs/02-deploying-an-app/001-app-deploy-helm/#tutorial-deploying-an-application-to-the-cloud-platform-with-helm) to the 'non-production cluster' using Helm.
 - You have created an [ECR repository](TODO) (docs coming soon)

### Creating a Service Account for CircleCI
As part of the CircleCI deployment pipeline, CircleCI will need to authenticate with the Kubernetes cluster. In order to do so, Kubernetes uses [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/). Service Accounts provide an identity for processes that run in a cluster allowing the process to access the API server.

A Service Account is created in the [namespace creation github repository](https://github.com/ministryofjustice/cloud-platform-environments/tree/master/namespaces).
```bash
  $ kubectl get serviceaccounts --namespace $ns
  NAME       SECRETS   AGE
  circleci   1         3h
  
  $ kubectl get serviceaccounts/circleci --namespace reference-app -o yaml
  apiVersion: v1
  kind: ServiceAccount
  ...
  secrets:
  - name: circleci-token-prlkp
  
  $ kubectl get secrets --namespace $ns
  NAME                   TYPE                                  DATA      AGE
  circleci-token-prlkp   kubernetes.io/service-account-token   3         3h
  
  $ kubectl get secrets/circleci-token-prlkp --namespace $ns -o yaml
  ...
  namespace: cm..cA==
  token: ZX...EE=
```

### Migration to CircleCI 2.0
We are using CircleCI 2.0 if you already have a CircleCI ```circle.yml``` please [migrate](https://circleci.com/docs/2.0/migration/) your project to 2.0.

### Linking Repository to CircleCI
MoJ has as an account with CircleCI, please login to [CircleCI](https://circleci.com/dashboard) using GitHub credentials. Select project, and if config.yml is in the repo CircleCI will build and run tests.

### Adding the config.yml
CircleCI uses a YAML file to identify how you want your testing environment set up and what tests you want to run. On CircleCI 2.0, this file must be called ```config.yml``` and must be in a hidden folder called ```.circleci``` .

### Add variables to CircleCI
From Builds click the cog and select Enviroment Variables under Build Settings. The variables needed to add are.
```
- AWS_DEFAULT_REGION (vars generated if specific resources like ECR/S3 are needed)
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- ECR_ENDPOINT
- CA_CERT (used by k8s internally, read with 'see above kubectl get secrets/ca')
- CLUSTER_NAME (eg non-production.k8s.integration.dsd.io)
- TOKEN (k8s auth token generated when building the namespace, 'kubectl get secrets/circleci-token | base64 -d')
```

### Creating the config.yml
[Tutorial](https://circleci.com/docs/2.0/tutorials/) on creating a config.yml file. As long as you are building a Docker image you can configure Circle however you wish. A sample worker image build is described in [Dockerfile-worker](https://github.com/ministryofjustice/cloud-platform-reference-app/blob/master/Dockerfile-worker).
The only extra code you will need to add is in Upload to ECR and Deploy to Kubernetes: 
- Upload to ECR
```bash
deploy:
    name: Push application Docker image
    environment:
    command: |
    login="$(aws ecr get-login)"
    ${login}

    docker tag app "${ECR_ENDPOINT}/${CIRCLE_PROJECT_REPONAME}:${CIRCLE_SHA1}"
    docker push "${ECR_ENDPOINT}/${CIRCLE_PROJECT_REPONAME}:${CIRCLE_SHA1}"

    if [ "${CIRCLE_BRANCH}" == "master" ]; then
        docker tag app "${ECR_ENDPOINT}/${CIRCLE_PROJECT_REPONAME}:latest"
        docker push "${ECR_ENDPOINT}/${CIRCLE_PROJECT_REPONAME}:latest"
    fi
```
- Deploy to Kubernetes
```bash
deploy:
    if [ "${CIRCLE_BRANCH}" == "master" ]; then
        kubectl delete job django-app-circleci-db-migration
        helm upgrade django-app-circleci ./helm_deploy/django-app/. \
            --tiller-namespace=cloudplatforms-reference-app \
            --namespace=cloudplatforms-reference-app \
            --set image.repository="${ECR_ENDPOINT}/${CIRCLE_PROJECT_REPONAME}" \
            --set image.tag="latest" \
            --set deploy.host="${APPLICATION_HOST_URL}" \
            --set replicaCount="1" \
            --install \
            --debug
```
The principles behind the 'Authenticate to cluster' step are described in https://ministryofjustice.github.io/cloud-platform-user-docs/01-getting-started/002-authenticate/#authenticating-to-the-cluster
