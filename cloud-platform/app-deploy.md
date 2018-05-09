---
category: cloud-platform
expires: 2018-01-31
---
# Deploying an application to the Cloud Platform

## Overview

The aim of this guide is to walkthrough the process of deploying an application into the Cloud Platform.

This guide uses a pre-configured application as an example of how to deploy your own.

## Prerequisites

This guide assumes the following:

* Kubectl is installed and configured.
* Docker is installed and configured.
* Authentication with the cluster has been established.
* An environment has been created within the Cloud Platform.
* Access to AWS, with ECR upload permissions.
* AWS CLI configured with account credentials.

If you are not deploying your own application and would like to deploy the example application, clone the following repo:

[https://github.com/ministryofjustice/cloud-platform-demo-app](https://github.com/ministryofjustice/cloud-platform-demo-app)

## Pushing application to ECR

To deploy an application to the Cloud Platform, firstly the application image needs to be retrievable from a repository.

Amazon's ECR, within ECS is where all of the application images used by the Cloud Platform are stored.

### Creating or choosing a repository

On the ECR start page you will have the option create a new repository, or to search for an existing one.

Create a new repository, and name it something relevant:

![Image](images/create-ecr-repo.png)

After confirmation that the repository was successfully created, ignore the list of commands and click **Done**, at the bottom of the page.

### Authenticating with the repository

Select your newly created repository from the displayed list.

Within the repository, you will see the following section with values unique to you:

![Image](images/repo-values.png)

Click the **View Push Commands** button.

You will then be presented with a list of preconfigured terminal commands for you to run.

Copy the first command. You will need to add `--profile yourAWSProfile` if the AWS account you're pushing to is not the default in your CLI:

`aws ecr get-login --profile mojdsd --no-include-email --region eu-west-1`

Execute the command, then proceed to execute the Docker login command provided in the Terminal.

`Login Succeeded` will confirm you have been authenticated with the repository.

### Pushing the Docker image to the repository

Ensure the Docker image for your application has been built and is stored locally on your machine.

Now we need to tag the image with the tag provided by ECR, so I can be pushed into the correct repository.

View the **Push Commands** again, and modify the fourth command provided, ensuring the first tag is the one currently used on your machine:

`docker tag cloud-platform-demo-app:latest 926803513772.dkr.ecr.us-west-1.amazonaws.com/cloud-platform-demo-app:latest`

Finish by running the fifth command provided, to push the image to your repository.

`docker push 926803513772.dkr.ecr.us-west-1.amazonaws.com/cloud-platform-demo-app:latest`

A quick refresh of ECR, and your image should now be displayed.

## Configuring deployment files

To deploy an application to the Cloud Platform, a number of deployment files must first be configured.

These deployment files will make reference to your application's Docker image and handle configurations, such as ports and host resolving.

This guide will analyse the deployment files for `Cloud-Platform-Demo-App`.

The deployment files can be used as a basic template for your application, and can be found at:

[https://github.com/ministryofjustice/cloud-platform-demo-app](https://github.com/ministryofjustice/cloud-platform-demo-app)

### Deployment

Deployment files are used to specify core information about an application that is being deployed to the Cloud Platform.

See the contents of the `deployment.yml` file below:

```Yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cp-demo-app
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: cp-demo-app
    spec:
      containers:
      - name: cp-demo-app
        image: 926803513772.dkr.ecr.us-west-1.amazonaws.com/cloud-platform-demo-app:latest
        ports:
        - containerPort: 80
```

If you are using the file as a template for your own application, replace the `cp-demo-app` tags with ones suited to your application.

For the value of the `image:` key, you will see the Repository URI, which has been provided by ECR.

If you are using the file as a template for your own application, replace the value of `image:` with the Repository URI of your application, found in ECR.

### Service

Service files are used to specify port and protocol information for your application and are also used to bundle together the set of pods created by the deployment.

See the contents of the `service.yml` file below:

```Yaml
kind: Service
apiVersion: v1
metadata:
  name: cp-app-demo-svc
  labels:
    app: cp-app-demo-svc
spec:
  ports:
  - port: 80
    name: http
    targetPort: 80
  selector:
    app: cp-demo-app
```

If you are using the file as a template for your own application, replace the `cp-demo-app-svc` tags with ones suited to your application.

Also, ensure the `selector:` app tag is the same as specified in the `deployment.yml`.

### Ingress

Ingress files are to use to define external access to the application.

See the contents of the `ingress.yml` file below:

```Yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cp-demo-app-ing
spec:
  rules:
  - host: cp-demo-app.integration.dsd.io
    http:
      paths:
      - path: /
        backend:
          serviceName: cp-demo-app-svc
          servicePort: 80
```

If you are using the file as a template for your own application, replace the `cp-demo-app-ing` tag with one suited to your application.

Also, ensure the `serviceName:` tag is the same as specified in the `service.yml`.

## Configuring secrets

The following is an example of encoding (configuring) aws access-key credentials in your deployment.
(See https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables
for detailed information regarding providing base64 values in secret objects to Kuberenetes pods)

Create your AWS Credentials access key (making a note of the aws_access_key_id and aws_secret_access_key)

put the aws key secret and id into a file (in this example demo-v9.k8s) as follows ('thekey' referencing the aws_access_key_id
and 'thesecret' referencing the aws_secret_access_key):

```
apiVersion: v1

kind: Secret

metadata:

  name: demosecret-v9

type: Opaque

data:

  thekey: AKIAFTKSAW15HJLOGD

  thesecret: g8hjpmhvgfhk4547gfdshhjj

```

create secret generic secret from that file:
```$kubectl -n demo-app create secret generic demosecret-v9 --from-file=demosecret-v9=demo-v9.k8s```

'demo-app' being the namespace, 'demosecret-v9' referencing the 'name' in demo-v9.k8s and referencing the file 'demo-v9.k8s'
this will give the following screen output


```
secret "demosecret-v9" created
```
To view the output of your secret:

```
$kubectl -o json -n demo-app get secret demosecret-v9

```

This will output something like the following:

```
{
    "apiVersion": "v1",
    "data": {
        "demosecret-v9": "YXBpVmVyc2lvbjogdjEKa2luZDogU2VjcmV0Cm1ldGFkYXRhOgogIG5hbWU6IGRlbW9zZWNyZXQtdjkKdHlwZTogT3BhcXVlCmRhdGE6CiAgdGhla2V5OiBBS0lBRlRLU0FXMTVISkxPR0QKICB0aGVzZWNyZXQ6IGc4aGpwbWh2Z2ZoazQ1NDdnZmRzaGhqago="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2018-05-09T15:17:50Z",
        "name": "demosecret-v9",
        "namespace": "demo-app",
        "resourceVersion": "3021923",
        "selfLink": "/api/v1/namespaces/demo-app/secrets/demosecret-v9",
        "uid": "2315be2f-539c-11e8-8ff4-0aecd97fec6e"
    },
    "type": "Opaque"
}

```
Using the 'data' output from above, issue the following command (to validate and verify that it is the same as you originally inputted into the demo-v9.k8s file above):

```
$ base64 -D
YXBpVmVyc2lvbjogdjEKa2luZDogU2VjcmV0Cm1ldGFkYXRhOgogIG5hbWU6IGRlbW9zZWNyZXQtdjkKdHlwZTogT3BhcXVlCmRhdGE6CiAgdGhla2V5OiBBS0lBRlRLU0FXMTVISkxPR0QKICB0aGVzZWNyZXQ6IGc4aGpwbWh2Z2ZoazQ1NDdnZmRzaGhqago=

```
ctrl+d to exit

This will return similar to the following output:

```
apiVersion: v1
kind: Secret
metadata:
  name: demosecret-v9
type: Opaque
data:
  thekey: AKIAFTKSAW15HJLOGD
  thesecret: g8hjpmhvgfhk4547gfdshhjj

```
Add the AWS_ACCESS_KEY_ID referencing 'thekey' and AWS_SECRET_ACCESS_KEY referencing 'thesecret (as previously set) to the containers env in deployment-files/deployment.yaml

```
    spec:
      containers:
        - name: django-demo-container
          image: 926803513772.dkr.ecr.eu-west-1.amazonaws.com/cloud-platform-reference-app:django
          ports:
            - containerPort: 8000
          env:
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: demosecret-v9
                  key: thekey
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: demosecret-v9
                  key: thesecret

```

As Base64 is an encoding algorithm that merely presents data in an alternative format, the data IS NOT encrypted. The output of 'base64 -D' 
needs to be put into an git-crypted file that is referenced by the '.gitattribute' file. 

Just to be safe issue the command:

```
git-crypt status

```

to see what is (and isn't encrypted)

For more information regarding git-crypt see  https://github.com/ministryofjustice/cloud-platform-user-docs/blob/076be35f6f1826f4250b76317b6535734a2c095e/getting-started/git-crypt-setup.md

## Deploying application to the cluster

With all of the deployment files configured and saved locally, you can now deploy your application to the Cloud Platform.

Start by verifying you are connected to the environment you created:

`kubectl cluster-info`

Now ensure that the environment is empty by running each of the commands, `No resources found.` should be returned for all of them:

* `kubectl get pods`
* `kubectl get deployments`
* `kubectl get services` - ClusterIP should be returned.
* `kubectl get ingress`

With the environment empty, deploy the application by running the following command, that points to the `deployment-files` directory, in which the deployment files are stored:

`kubectl create -f deployment-files`

Confirm the deployment with:

`kubectl get pods`

## Interacting with the application

With the application deployed into the Cloud Platform, there are a few ways of managing it:

* **View pods** - `kubectl get pods`
* **Check host** - `kubectl get ing`
* **Delete application** - `kubectl delete -f deployment-files`
* **Shell into container** - `kubectl exec -it pod-name -- /bin/bash`
