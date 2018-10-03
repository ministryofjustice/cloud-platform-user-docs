---
category: cloud-platform
expires: 2019-01-01
---
# Using the Cloud Platform Prometheus, AlertManager and Grafana 
## Introduction
Prometheus, AlertManager and Grafana have been installed on Live-0 to ensure the reliability and availability of the Cloud Platform. This document will briefly outline how to access the monitoring tools and where to find further information. 

## What is Prometheus

Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. The Cloud Platform uses [Prometheus Operator from CoreOS](https://github.com/coreos/prometheus-operator) which allows a number of Prometheus instances to be installed on a cluster.

Prometheus scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs. It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts. Grafana or other API consumers can be used to visualize the collected data.

## What is AlertManager

The Alertmanager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver integration such PagerDuty. It also takes care of silencing and inhibition of alerts.

## What is Grafana

Grafana allows you to query, visualize, alert on and understand your metrics no matter where they are stored. Create, explore, and share dashboards with your team and foster a data driven culture.

## How to access monitoring tools

All links provided below require you to authenticate with your Github account. 

Prometheus: [https://prometheus.apps.cloud-platform-live-0.k8s.integration.dsd.io](https://prometheus.apps.cloud-platform-live-0.k8s.integration.dsd.io)

AlertManager: [https://alertmanager.apps.cloud-platform-live-0.k8s.integration.dsd.io](https://alertmanager.apps.cloud-platform-live-0.k8s.integration.dsd.io/)

Grafana: [https://grafana.apps.cloud-platform-live-0.k8s.integration.dsd.io](https://grafana.apps.cloud-platform-live-0.k8s.integration.dsd.io)


## Further documentation

[Prometheus querying](https://prometheus.io/docs/prometheus/latest/querying/basics)

[AlertManager](https://prometheus.io/docs/alerting/alertmanager)

[Architecture](https://prometheus.io/docs/introduction/overview/#architecture)