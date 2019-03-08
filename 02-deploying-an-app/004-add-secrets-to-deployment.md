---
category: cloud-platform
expires: 2018-01-31
---
# Adding a secret to an application

## Overview

The aim of this guide is to walkthrough the process of adding a secret (in this example for aws access-key credentials) to a previously deployed application in the Cloud Platform.

## Prerequisites

This guide assumes the following:

* You have previously set up an env. See [Creating a Cloud Platform Environment]({{ "/01-getting-started/002-env-create" | relative_url }})
* You have previously deployed your application. See [Deploying an application to the Cloud-Platform]({{ "/02-deploying-an-app/001-app-deploy" | relative_url }})
* Check your deployment is running. See [Interacting with the application]({{ "/02-deploying-an-app/001-app-deploy/#interacting-with-the-application" | relative_url }})

## Configuring secrets

The following is an example of encoding (configuring) aws access-key credentials in your deployment.
See [kuberenetes using secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

for detailed information regarding providing base64 values in secret objects to Kuberenetes pods)

Create your AWS Credentials access key (making a note of the aws_access_key_id and aws_secret_access_key)

### base64-encode your secret as follows:

In this example  aws_access_key_id is 'AKIAFTKSAW15HJLOGD'. Issue the following command to base64-encode:

```
echo -n 'AKIAFTKSAW15HJLOGD' | base64 -b0
```

This will return the encoded id 'QUtJQUZUS1NBVzE1SEpMT0dE'

In this example the is aws_secret_access_key 'g8hjpmhvgfhk4547gfdshhjj'. Issue the following command to base64-encode:

```
echo -n 'QUtJQUZUS1NBVzE1SEpMT0dE' | base64 -b0
```

This will return the encoded secret 'UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF'

## Creating the secret

Create a secrets.yaml file similar to:

```Yaml
apiVersion: v1
kind: Secret
metadata:
  name: demosecret
type: Opaque
data:
  aws_access_key_id: QUtJQUZUS1NBVzE1SEpMT0dE
  aws_secret_access_key: UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF
```
issue the following command:

```
$ kubectl apply -f secrets.yaml
secret "demosecret" created
```

To see the secrets:

```
$ kubectl get secrets
NAME                                          TYPE                                  DATA      AGE
calico-zebu-external-dns-token-pldjb          kubernetes.io/service-account-token   3         16d
dandy-bumblebee-nginx-ingress-token-bspl6     kubernetes.io/service-account-token   3         14d
default-token-hz7z7                           kubernetes.io/service-account-token   3         26d
demosecret                                    Opaque                                2         5d
```

Decoding the Secret
Secrets can be retrieved via the kubectl get secret command. For example, to retrieve the secret you created:

```
$ kubectl get secret demosecret  -o yaml
apiVersion: v1
data:
  aws_access_key_id: dGVzdCBrZXk=
  aws_secret_access_key: dGVzdCBrZXk=
kind: Secret
metadata:
  creationTimestamp: 2018-05-15T12:24:33Z
  name: demosecret
```

Add the AWS_ACCESS_KEY_ID referencing 'aws_access_key_id' and AWS_SECRET_ACCESS_KEY referencing 'aws_secret_access_key' (as previously set) to the containers env in `deployment-files/deployment.yaml`

```Yaml
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
                  name: demosecret
                  key: aws_access_key_id
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: demosecret
                  key: aws_secret_access_key
```
