---
category: cloud-platform
expires: 2018-01-31
---
# Creating a Cloud Platform Environment

## Aim

This is a guide to creating a non-production environment in our non-production Kubernetes cluster. This document is intended for any developers who are interested in creating a development or staging environments within Kubernetes.

## Description

[K8s-nonprod-environments repo](https://github.com/ministryofjustice/k8s-nonprod-environments)

This repo contains the necessary files to create a pipeline in AWS to create Kubernetes cluster namespaces and resources after a push has been made to the master branch of this repo.

This repo contains the following files and directories

#### Terraform

This directory contains terraform resources to create AWS pipeline for creation of kubernetes namespace and resource creation.

#### Buildspec

The `buildspec.yaml` file contains codebuild specification that will create resources defined in the namespaces.

#### Namespace

The namespaces contains subdirectories named after each of the desired namespaces you want to create.

You will be working within the namespace directory to create your environment.

### GitHub directory structure

**k8s-non-prod-environment**

![Image](images/image4.png)

This is the root of the repo, containing Terraform and Namespace directory

**/namespaces**

![Image](images/image5.png)

This is the Namespace directory, this is where you will create a directory for your service in the format `$servicename-$env`   
example: myapp-dev

**/$Servicename-$env**

![Image](images/image6.png)

When you create your `$servicename-$env` directory for your service. You will need to create two files within it. `Namespaces.yaml` and `$servicename-$env-admin-role.yaml`

#### Instructions

![Image](images/image2.png)

**1)** Git clone the repo onto your local machine and create a new branch to make your changes. (Using terminal)

```
  #git clone the repo onto your local machine.
  $ git clone git@github.com:ministryofjustice/k8s-nonprod-environments.git

  # change directory into the k8s-non prod repo
  cd k8s-nonprod-environments/

  #create new branch in the repo, my-app is used as an example, you can call it something descriptive.
  $ git checkout -b my-app

  #you can confirm you are using the branch you made using the command.
  $ git branch -a

  #Current branch you are currently using will be indicated by the *.

    master
  * my-app
    remotes/origin/HEAD -> origin/master
    remotes/origin/add-git-crypt
```


**2)** Now we need to amend the namespaces directory in the root of repo and create an directory within it.

```
#we will need to change directories and change into the namespaces directory.
cd namespaces/

#we shall now be in our namespaces directory but we can confirm by the following command.
pwd

#output should show that we are in the namespaces directory. This will all be dependent on where you have chosen to git clone the repo. In this example k8s-nonprod was cloned into the Git directory which is located in Documents.

$ pwd
/Users/<username>/Desktop/Git/k8s-nonprod-environments/namespaces

#Now we must create the directory for our service, the name of the directory we create should be in the format $servicename-$env
$ mkdir myapp-dev

#Now that we have created our namespaces subdirectory $servicename-env, lets now change into this directory.
$ cd myapp-dev
```


**3)** Now we will need to create two files within our `$servicename-$env` directory.  `Namespaces.yaml` and `$servicename-$env-admin-role.yaml`. The structures of both files are shown below.


**k8s-non-prod-environment/namespaces/$servicename-$env/namespace.yaml**

```
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-dev
  labels:
    name: myapp-dev
```

**Namespace:**

Namespaces is a mechanism in Kubernetes that will essentially create your environment. 

Kubernetes namespace creates a logical cluster within our cluster that provides isolation from any other namespace.

Using a Kubernetes namespace could isolate namespaces for different environments in the same cluster. Providing us with flexibility of creating segregated environments for different.

**Namespace.yaml**

```
apiVersion: 
kind: 
metadata:
      name: This is where you will define your $servicename-$env
      labels:
           name: Also define your $servicename-$env
```

[https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/)

**k8s-non-prod-environment/namespaces/$servicename-$env/$service-$name-admin-role.yaml**

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: myapp-dev-admins
  namespace: myapp-dev
subjects:
  - kind: Group
    name: "github:$yourTeam"
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

**Rolebinding:**

We will also create a Rolebinding resouce that will provide us with access policies to the namespace we have created in the cluster.

A rolebinding resource grants the permissions defined in a role to a user or set of users. A role can be another resource we can create but in this instance we will reference (roleRef) a Kubernetes default role "ClusterRole - admin" .

This Rolebinding resource references the ClusterRole to provide ClusterRole admin access to a set of users which is defined under subjects. In this case, the `$yourTeam` github group will have admin access to any resources within the namespace "myapp dev".

`$servicename-$env-admin-role.yaml`

Example of yaml above, i will explain parts that need to be changed.

```
kind: 
apiVersion:
metadata:
     name:
     namespace: Define your namespace as created in namespace.yaml, this is where its mapped.
subjects:
     kind: 
     name: This is where you specify your Github team, as format shown above "github:$yourTeam"
     apiGroup:
roleRef: 
     kind: 
     name:
     apiGroup:
```

[https://kubernetes.io/docs/admin/authorization/rbac/](https://kubernetes.io/docs/admin/authorization/rbac/)

```
within our $servicename-$env directory, you will need to create the two files shown above. You can use any file editor of your choice (nano, vim).

$ cd myapp-dev/       								# Make sure you are in your service directory.
$ vi namespace.yaml   								#create the namespace.yaml and use guide above to help structure file.
$ vi myapp-dev-admin-role.yaml 						#once completed do the same for admin-role file.
$ ls                            					#list files in $Servicename-$dev directory, to see both files have been saved.
myapp-dev-admin-role.yaml	namespace.yaml   		#output after running list command.

##Remember these are Yaml files, so make sure syntax is correct. If syntax is incorrect then file may not build properly, you can use a online validator.
#https://codebeautify.org/yaml-validator

```


**4)** After both files are created. You are ready to upload your branch to our github repo. You will need to add the files, commit them and then set a new upstream branch.

```
# Add all new files from your current directory to staging.
$ git add

# Can run the command git status to check what changes are to be commited.
$ git status

# output will show files that are to be commited in green.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   namespaces/myapp-dev/myapp-dev-admin-role.yaml
	new file:   namespaces/myapp-dev/namespace.yaml

#once your happy with the files to be uploaded, you can now run git commit to commit the files and -m parameter to add message to your commit.
$ git commit -m "adding namespace.yaml and admin-roles for myapp to create dev environment"

#Should get output reflecting files added.

2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 namespaces/myapp-dev/myapp-dev-admin-role.yaml
 create mode 100644 namespaces/myapp-dev/namespace.yam

#Now you are ready to upload your branch to the repo. myapp being your branch name.
git push --set-upstream origin myapp

#Now you can go to the repo in the GITHUB GUI and make a pull request to merge your branch with master.
```


**5)** Now that you have uploaded your branch, go to the Github GUI and select your branch. Once you have done this, you may make a pull request to merge to master.
![Image](images/image3.png)
