---
category: cloud-platform
expires: 2019-07-01
---
# How do I create my own alerts?

## Overview
Alertmanager allows you define your own alert conditions based on [Prometheus expression language](https://prometheus.io/docs/prometheus/latest/querying/basics) expressions. 

The aim of this document is to provide you with the necessary information to create and send application specific alerts to a Slack channel of your choosing.

## Prerequisites
This guide assumes the following:

* You have [created a namespace for your application]({{ "/01-getting-started/002-env-create" | relative_url }})
* Deployed at least one pod in your namespace 

## Creating a slack webhook and amend Alertmanager
This step requires the Cloud Platform team to create a receiver in [Alertmanager](https://github.com/ministryofjustice/cloud-platform-infrastructure/blob/master/terraform/cloud-platform-components/templates/prometheus-operator.yaml.tpl#L115) and a [Slack webhook](https://api.slack.com/incoming-webhooks).

Use the #ask-cloud-platform Slack channel to request a new alert route in Alertmanager. The team will need the following information:
  
- namespace name
- team name
- application name
- slack channel

The team will provide you with a `value` that'll need to be defined in the next step. Please copy it to your clipboard. 

## Create a PrometheusRule
A `PrometheusRule` is a custom resource that defines your triggered alert. This file will contain the alert name, promql expression and time of check. 

To create your own custom alert you'll need to fill in the template below and deploy it to your namespace (tip: you can check rules in your namespace by running `kubectl get prometheusrule -n <namespace>`). 

- Create a file called `prometheus-custom-rules-<application_name>.yaml`
- Copy in the below and replace the required values in brackets. The `<team_name>` value under `severity` is the value you were passed earlier. 
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
        severity: <team_name>
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
  namespace: test-namespace
  labels:
    role: alert-rules
  name: prometheus-custom-rules-my-application
spec:
  groups:
  - name: node.rules
    rules:
    - alert: CPU-High
      expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: cp-team
      annotations:
        message: This device's CPU usage has exceeded the threshold with a value of {{ $value }}. Instance {{ $labels.instance }} CPU usage is dangerously high
        runbook_url: http://link-to-support-docs.website
```

The `alert` name, `message` and `runbook_url` annotations will be sent to the Slack channel when the rule has been triggered. 

You can view the applied rules with the following command:

```sh 
kubectl -n <namespace> describe prometheusrules prometheus-custom-rules-<application_name>
```

## PrometheusRule examples
If you're struggling for ideas on how and which alerts to setup, please see some examples [here](https://github.com/ministryofjustice/cloud-platform-infrastructure/blob/master/terraform/cloud-platform-components/resources/prometheusrule-examples/application-alerts.yaml).

## Further reading
- [Prometheus Operator - Getting Started](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md)
- [Alerting](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/alerting.md)
