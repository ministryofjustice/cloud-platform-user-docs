---
category: cloud-platform
expires: 2019-01-01
---
# Kubernetes quick reference

This document acts as a quick reference to `kubectl`, listing some of the most common operations.

The examples here only address the most basic approach to these operations. For more options, please refer to the command-line help of `kubectl` subcommands.

There is also a more detailed [cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) in the official kubernetes documentation.

## Table of contents
{:toc}

#### Inspecting running instances of the application
To list running `Pods`:
```
$ kubectl -n <environment> get pods
```
To view details for a `Pod`:
```
$ kubectl -n <environment> describe pod <pod>
```
#### Viewing logs
To access the logs of a running container:
```
$ kubectl -n <environment> logs <pod>
```

#### Viewing kubernetes events
To see kubernetes events, which can help debugging:
```
$ kubectl -n <environment> get events
```

#### Container shell
You can get a shell inside a running container:
```
$ kubectl -n <environment> exec -it <pod> sh
```
https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/


#### Pod port-forwarding
To forward port `5000` on `localhost` to port `5001` in the `Pod`:
```
$ kubectl -n <environment> port-forward <pod> 5000:5001
```
https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/
