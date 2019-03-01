---
category: cloud-platform
expires: 2019-07-01
---
# Creating your own custom alerts

## Overview
Alertmanager allows you define your own alert conditions based on [Prometheus expression language](https://prometheus.io/docs/prometheus/latest/querying/basics) expressions. 

The aim of this document is to provide you with the necessary information to create and send application specific alerts to a Slack channel of your choosing.

## Prerequisites
This guide assumes the following:

* You have [created a namespace for your application]({{ "/01-getting-started/002-env-create" | relative_url }})

## Creating a slack webhook and amend Alertmanager
This step requires the Cloud Platform team to create a receiver in [Alertmanager](https://github.com/ministryofjustice/cloud-platform-infrastructure/blob/master/terraform/cloud-platform-components/templates/prometheus-operator.yaml.tpl#L115) and a [Slack webhook](https://api.slack.com/incoming-webhooks).

Create a ticket to request a new alert route in Alertmanager. The team will need the following information:
  
- namespace name
- team name
- application name
- slack channel

The team will provide you with a "`custom severity level`" that'll need to be defined in the next step. Please copy it to your clipboard. 

## Create a PrometheusRule
A `PrometheusRule` is a custom resource that defines your triggered alert. This file will contain the alert name, promql expression and time of check. 

To create your own custom alert you'll need to fill in the template below and deploy it to your namespace (tip: you can check rules in your namespace by running `kubectl get prometheusrule -n <namespace>`). 

- Create a file called `prometheus-custom-rules-<application_name>.yaml`
- Copy in the template below and replace the bracket values, specifying the requirements of your alert. The `<custom_severity_level>` field is the value you were passed earlier. 

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  namespace: <namespace>
  labels:
    role: alert-rules
  name: prometheus-custom-rules-<application_name>
spec:
  groups:
  - name: application-rules
    rules:
    - alert: <alert_name>
      expr: <alert_query>
      for: <check_time_length>
      labels:
        severity: <custom_severity_level>
      annotations:
        message: <alert_message> 
        runbook_url: <http://my-support-docs>
```
- Run `kubectl apply -f prometheus-custom-rules-<application_name>.yaml -n <namespace>`

A working example of this would be:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  creationTimestamp: null
  namespace: monitoring
  labels:
    role: alert-rules
  name: prometheus-custom-rules-my-application
spec:
  groups:
  - name: node.rules
    rules:
    - alert: Quota-Exceeded
      expr: 100 * kube_resourcequota{job="kube-state-metrics",type="used",namespace="monitoring"} / ignoring(instance, job, type) (kube_resourcequota{job="kube-state-metrics",type="hard"} > 0) > 90
      for: 5m
      labels:
        severity: cp-team
      annotations:
        message: Namespace {{ $labels.namespace }} is using {{ printf "%0.0f" $value}}% of its {{ $labels.resource }} quota.
        runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubequotaexceeded
```

The `alert` name, `message` and `runbook_url` annotations will be sent to the Slack channel when the rule has been triggered. 

You can view the applied rules with the following command:

```sh 
kubectl -n <namespace> describe prometheusrules prometheus-custom-rules-<application_name>
```

## PrometheusRule examples
If you're struggling for ideas on how and which alerts to setup, please see some examples [here](https://github.com/ministryofjustice/cloud-platform-infrastructure/blob/master/terraform/cloud-platform-components/resources/prometheusrule-examples/application-alerts.yaml).

## Application Metrics

Prometheus collects metrics from monitored targets by scraping metrics HTTP endpoints on these targets. There are two ways to create a metrics endpoint. The first is when the metrics endpoint is embedded within the application referred to as `instrumentation`. The second is when the metrics endpoint is part of a deployed process that bridges the gap between Prometheus and systems that do not export metrics in the prometheus format, this is called an `exporter`.

The following exporters are installed as part of the Cloud Platform cluster build:

- kubeEtcd
- kubeApiServer
- kubeDns
- kubeControllerManager
- kubeScheduler
- kube-state-metrics
- nodeExporter
- kubelet

Click [here](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exporters.md) for a list of exporters and client libraries listed on the official Prometheus Github repo.


### Instrumentation of The Cloud-Platform Reference Application
The [Cloud Platform Reference Application](https://github.com/ministryofjustice/cloud-platform-reference-app) has `instrumentation` setup using [django-prometheus](https://github.com/korfuri/django-prometheus). The default install steps were followed which created a metrics endpoint.

See screenshot below of how the metrics endpoint looks on a browser:

![Image of metrics](https://raw.githubusercontent.com/ministryofjustice/cloud-platform-user-docs/master/images/metrics_endpoint.png)


### Create a Service to expose Pods

Scraping an exporter or separate metrics port requires a service that targets the Pod(s) of the exporter or application.

Example:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-app-service
  labels:
    app: my-app 
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 8000
    name: metrics
  type: NodePort
```

### Create a Service-Monitor
A `ServiceMonitor` is a resource the Prometheus Operator introduces for Kubernetes that describes the set of targets to be monitored by Prometheus

Service Monitor Architecture
![Image of Service-Monitor Architecture](https://raw.githubusercontent.com/ministryofjustice/cloud-platform-user-docs/master/images/service-monitor-arch.png)


Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 15s
```

More detailed information about service-monitors can be found [here](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/running-exporters.md)

### NetworkPolicy for Monitor Namespace

By default, all connections from outside a namespace are blocked, therefore a network policy is required for the `monitoring` namespace to be able to connect into an application namespace to scrape the metrics endpoint.

Example:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata: 
  name: allow-prometheus-scraping
  namespace: my-app-namespace
spec:
  podSelector:
    matchLabels: 
      app: my-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector: 
        matchLabels: 
          component: monitoring
```

## Further reading
- [Prometheus Operator - Getting Started](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md)
- [Alerting](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/alerting.md)
