# Cloud platform user guide

## Overview

The Ministry of Justice cloud platform is a platform for hosting services developed for the Ministry of Justice. The platform manages the infrastructure that your application runs on and provides tooling for teams to deploy and manage their applications on that infrastructure.

### Who is the platform for

The platform is for teams working anywhere in the Ministry of Justice that need to run software applications. To get started you need to be part of a team in the Ministry of Justice GitHub organisation. Once you are, you can start to deploy applications using the guides below. 

### What do we currently support

Users can create non-production and production environments and get access to the Kubernetes API for those environments.

### Help and support

For help or support, drop into the MoJ Digital slack channel `#ask-cloud-platform`, raise an issue on the [Ministry of Justice cloud-platform](https://github.com/ministryofjustice/cloud-platform) repo, or email the cloud platform team `platforms@digital.justice.gov.uk`.

## Getting started

{% assign getting_started_groups = site.pages
  | where: "01-getting-started", true %}

{% for article in getting_started_groups %}
  [{{ article.title }}]({{ article.url | relative_url }})
{% endfor %}

## Deploying an app

{% assign deploying_groups = site.pages
  | where: "02-deploying-an-app", true %}

{% for article in deploying_groups %}
  [{{ article.title }}]({{ article.url | relative_url }})
{% endfor %}

## Monitoring an app

{% assign monitoring_groups = site.pages
  | where: "03-monitoring-an-app", true %}

{% for article in monitoring_groups %}
  [{{ article.title }}]({{ article.url | relative_url }})
{% endfor %}

## Other topics

{% assign other_topics_groups = site.pages
  | where: "99-other-topics", true %}

{% for article in other_topics_groups %}
  [{{ article.title }}]({{ article.url | relative_url }})
{% endfor %}
