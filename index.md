# Cloud platform user guide

## Overview

The Ministry of Justice cloud platform is a platform for hosting services developed for the Ministry of Justice. The platform manages the infrastructure that your application runs on and provides tooling for teams to deploy and manage their applications on that infrastructure.

### Who is the platform for

The platform is for teams working anywhere in the Ministry of Justice that need to run software applications. If you are interested in becoming a user, drop into the MoJ Digital slack channel `#cloud-platform-users` or email the cloud platform team `platforms@digital.justice.gov.uk`.

### What do we currently support

Users can create non-production environments and get access to the Kubernetes API for those environments.  

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

## Other topics

{% assign other_topics_groups = site.pages
  | where: "03-other-topics", true %}

{% for article in other_topics_groups %}
  [{{ article.title }}]({{ article.url | relative_url }})
{% endfor %}
