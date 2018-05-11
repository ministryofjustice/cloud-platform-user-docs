---
category: cloud-platform
expires: 2018-01-31
---
# Adding a secret to an application

## Overview

The aim of this guide is to walkthrough the process of adding a secret (in this example for aws access-key credentials) to a previously deployed application in the Cloud Platform.

## Prerequisites

This guide assumes the following:

* You have previously set up an env. See [Creating-a-Cloud-Platform-Environment](https://github.com/ministryofjustice/cloud-platform-user-docs/blob/master/cloud-platform/env-create.md/#using-secrets-as-environment-variables/#creating-a-cloud-platform-environment)
* You have previously deployed your application. See [Deploying-an-application-to-the-Cloud-Platform](https://github.com/ministryofjustice/cloud-platform-user-docs/blob/master/cloud-platform/app-deploy.md/#deploying-an-application-to-the-cloud-platform)
* Check your deployment is running. See [Interacting-with-the-application](https://github.com/ministryofjustice/cloud-platform-user-docs/blob/master/cloud-platform/app-deploy.md/#interacting-with-the-application)

## Configuring secrets

The following is an example of encoding (configuring) aws access-key credentials in your deployment.
See [kuberenetes-using-secrets-as-environment-variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

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

 

put the encoded secret and id into a file (in this example demo-v9.k8s) as follows:

```Yaml
apiVersion: v1

kind: Secret

metadata:

  name: demosecret-v9

type: Opaque

data:

  aws_access_key_id: QUtJQUZUS1NBVzE1SEpMT0dE

  aws_secret_access_key: UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF
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
To exit the command 

```
ctrl+d 
```

This will return similar to the following output:

```
apiVersion: v1
kind: Secret
metadata:
  name: demosecret-v9
type: Opaque
data:
  aws_access_key_id: QUtJQUZUS1NBVzE1SEpMT0dE
  aws_secret_access_key: UVV0SlFVWlVTMU5CVnpFMVNFcE1UMGRF
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
                  name: demosecret-v9
                  key: aws_access_key_id
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: demosecret-v9
                  key: aws_secret_access_key
```

As Base64 is an encoding algorithm that merely presents data in an alternative format, the data IS NOT encrypted. The output of 'base64 -D'
needs to be put into an git-crypted file that is referenced by the '.gitattribute' file.

Just to be safe issue the command:

```
git-crypt status
```

to see what is (and isn't encrypted)

For more information regarding git-crypt see [git-crypt-setup](https://github.com/ministryofjustice/cloud-platform-user-docs/blob/076be35f6f1826f4250b76317b6535734a2c095e/getting-started/git-crypt-setup.md)

# Updating the cluster

* to preview the rolling update

```
kops rolling-update cluster
```

* to apply the rolling-update

```
kops rolling-update cluster --yes
```
* to update the cluster with the config from state store

```
kops update cluster <cluster-name> --yes --state=s3://<kops-state-store-link>
```
