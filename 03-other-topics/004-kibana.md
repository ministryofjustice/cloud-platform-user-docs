# How to accesss my log data
## Introduction
This document is intended to assist engineers in accessing application and system logs stored in a centralised Elasticsearch cluster. Using a combination of Fluentd, AWS's Elasticsearch cluster and Kibana the Cloud Platform collects, indexes and presents your application and system log data. All that's required is that your application writes to stdout. 

Production and non-production data will be indexed in seperate clusters. 
## Quick start
<insert kibana link>
## How to get my data into Elasticsearch
As long as your application is writing to stdout, it'll be collected by fluentd and passed to the Elasticsearch cluster. 
## How do I use Kibana?
<insert elasticsearch links>
