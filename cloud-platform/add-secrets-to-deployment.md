---
category: cloud-platform
expires: 2018-01-31
---
# Adding a secret to an application

## Overview

The aim of this guide is to walkthrough the process of adding a secret (in this example for aws access-key credentials) to a previously deployed application in the Cloud Platform.

## Prerequisites

This guide assumes the following:

* You have previously set up an env. See [Creating a Cloud Platform Environment](/cloud-platform/env-create)
* You have previously deployed your application. See [Deploying an application to the Cloud-Platform](/cloud-platform/app-deploy)
* Check your deployment is running. See [Interacting with the application](/cloud-platform/app-deploy/#interacting-with-the-application)

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

There are 2 ways of creating the secret:

1. create a generic secret - for Concourse's git-crypt key


2. Create the secret with a YAML file

### 1. create a generic secret - for Concourse's git-crypt key

This is Concourse-specific from a file. With this method output is base64d twice  

put the encoded secret and id into a file (in this example secret-demo.k8s, can be any text file in the format below) as follows:

```
apiVersion: v1

kind: Secret

metadata:

  name: demosecret

type: Opaque

data:

  aws_access_key_id: QUtJQUZUS1NBVzE1SEpMT0dE

  aws_secret_access_key: UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF
```

create secret generic secret from that file:
```$kubectl -n demo-app create secret generic demosecret --from-file=demosecret=secret-demo.k8s```

'demo-app' being the namespace, 'demosecret' referencing the 'name' in secret-demo.k8s and referencing the file 'secret-demo.k8s'
this will give the following screen output


```
secret "demosecret" created
```
To view the output of your secret:

```
$ kubectl get secret demosecret -n demo-app -o jsonpath="{.data.aws_secret_key}"
QUtJQUZUS1NBVzE1SEpMT0dE
```
This can then be piped directly into base64 to get the plaintext value:

```
$ kubectl get secret demosecret -n demo-app -o jsonpath="{.data.aws_secret_key}" | base64 --decode
AKIAFTKSAW15HJLOGD
```
Optional - to see more detail

```
$kubectl -o json -n demo-app get secret demosecret
```

This will output something like the following:

```
{
    "apiVersion": "v1",
    "data": {
        "demosecret": "YXBpVmVyc2lvbjogdjEKa2luZDogU2VjcmV0Cm1ldGFkYXRhOgogIG5hbWU6IGRlbW9zZWNyZXQtdjkKdHlwZTogT3BhcXVlCmRhdGE6CiAgdGhla2V5OiBBS0lBRlRLU0FXMTVISkxPR0QKICB0aGVzZWNyZXQ6IGc4aGpwbWh2Z2ZoazQ1NDdnZmRzaGhqago="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2018-05-09T15:17:50Z",
        "name": "demosecret",
        "namespace": "demo-app",
        "resourceVersion": "3021923",
        "selfLink": "/api/v1/namespaces/demo-app/secrets/demosecret",
        "uid": "2315be2f-539c-11e8-8ff4-0aecd97fec6e"
    },
    "type": "Opaque"
}
```
Using the 'data' output from above, issue the following command (to validate and verify that it is the same as you originally inputted into the secret-demo.k8s file above):

```
$ base64 -D
YXBpVmVyc2lvbjogdjEKa2luZDogU2VjcmV0Cm1ldGFkYXRhOgogIG5hbWU6IGRlbW9zZWNyZXQtdjkKdHlwZTogT3BhcXVlCmRhdGE6CiAgdGhla2V5OiBBS0lBRlRLU0FXMTVISkxPR0QKICB0aGVzZWNyZXQ6IGc4aGpwbWh2Z2ZoazQ1NDdnZmRzaGhqago=
```
To exit the base64 command (this will not log you out of your session) 

```
ctrl+d 
```

This will return similar to the following output:

```
apiVersion: v1
kind: Secret
metadata:
  name: demosecret
type: Opaque
data:
  aws_access_key_id: QUtJQUZUS1NBVzE1SEpMT0dE
  aws_secret_access_key: UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF
```
### 2. Create the secret with a YAML file

Create a yaml file similar to:

```
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

``` kubectl create -f demo.yaml ```

This will output:

``` secret "demosecret" created ```

To see the secrets:

``` kubectl get secrets ```

Will output similar to

```
NAME                                          TYPE                                  DATA      AGE
calico-zebu-external-dns-token-pldjb          kubernetes.io/service-account-token   3         16d
dandy-bumblebee-nginx-ingress-token-bspl6     kubernetes.io/service-account-token   3         14d
default-token-hz7z7                           kubernetes.io/service-account-token   3         26d
demosecret                                    Opaque                                2         5d
```

Decoding the Secret
Secrets can be retrieved via the kubectl get secret command. For example, to retrieve the secret you created:

``` kubectl get secret demosecret  -o yaml ```

output

```
apiVersion: v1
data:
  aws_access_key_id: dGVzdCBrZXk=
  aws_secret_access_key: dGVzdCBrZXk=
kind: Secret
metadata:
  creationTimestamp: 2018-05-15T12:24:33Z
  name: demosecret


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

As Base64 is an encoding algorithm that merely presents data in an alternative format, the data IS NOT encrypted. 
The output of 'base64 -D'needs to be put into an [git crypted](/getting-started/git-crypt-setup) file that is referenced by the '.gitattribute' file.

Just to be safe issue the command:


```
git-crypt status
```

to see what is (and isn't encrypted)
