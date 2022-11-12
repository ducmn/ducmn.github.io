---
layout: page
title: Snowplow on GCP
---

[Snowplow](https://snowplow.io/) is a highly customisable behavioural data platform, and blessed be that company since their code is open-source (amen).

For some reason, the only guide for implementing Snowplow on Google Cloud Platform [was written in 2019](https://www.simoahava.com/analytics/install-snowplow-on-the-google-cloud-platform/), so I think it's time for an update. Most of the content below was shamefully lifted directly from the aforementioned article.

# Pre-requisites
You should have your GCP account with billing enabled.

# Host your Snowplow JS tracker file somewhere
- Register in [Search Console](https://search.google.com/search-console/welcome) your prefered domain name that host the JS file; once done, you can delete the record that was used to verify
- Follow [this guide](https://docs.snowplow.io/docs/collecting-data/collecting-from-own-applications/javascript-trackers/javascript-tracker/self-hosting-the-javascript-tracker/self-hosting-the-javascript-tracker-on-google-cloud/)

# Enable the required services
Go ahead and switch on:
- Compute Engine API
- Cloud Pub/Sub API

# Install Google Cloud CLI
[It](https://cloud.google.com/sdk/docs/install) makes interacting with GCP easier.

# Service account
If you've used Compute Engine before, your should already a powerful service account set up for you. Otherwise, you can quickly set one up for yourself.

# Set up Pub/Sub topics
Create topics called "good", "bq-failed-inserts", "bq-types", "enriched-good", and "bad", though only the first four need subscriptions.

# Create the config files
Four files you'll need, at the minimum. They are:
- [Stream collector config](https://github.com/snowplow/stream-collector/blob/master/examples/config.pubsub.extended.hocon)
- [Enricher config](https://github.com/snowplow/enrich/blob/master/config/config.pubsub.minimal.hocon)
- [Loader config](https://github.com/snowplow-incubator/snowplow-bigquery-loader/blob/master/config/config.minimal.hocon)
- [Iglu resolver config](https://github.com/snowplow/enrich/blob/master/config/iglu_resolver.json)
Store them all in cloud storage, for you'll need them later.

# Create a HTTPS endpoint that connects to stream collector
## Create your stream collector template
- Go to Compute Engine section
- Create instance template
- Choose "Set access for each API"
- Enable Cloud Pub/Sub
- Under Firewall, select "Allow HTTP traffic"
- Expand "Advance options"
- Expand "Management"
- Under "Automation", fill in the script below:
    #! /bin/bash
    sudo apt-get update
    sudo apt-get -y install default-jre
    sudo apt-get -y install unzip
    sudo apt-get -y install wget
    wget "https://github.com/snowplow/stream-collector/releases/download/2.8.2/snowplow-stream-collector-google-pubsub-2.8.2.jar"
    gsutil cp gs://your-bucket/your-stream-collector-config .
    java -jar snowplow-stream-collector-google-pubsub-2.8.2.jar --config your-stream-collector-config
- Expand "Networking"
- In "Network tags", add `collector`
- Click "Create"
## Add firewall rule
- In VPC network section, go to "Firewall"
- In "Target tags", type `collector`
- In "Source IPv4 ranges", type `0.0.0.0/0`
- Tick TCP, then type `8080`
- Click "Create"
## Create a health check in Compute Engine
- Protocol: HTTP
- Port: 8080
- Request path: /health
- Check interval: 10 seconds
- Unhealthy threshold: 3 consecutive failures
## Create stream collector instance group
- In "Instance template", select the template you created
- In "Health check", select the health check you created
- Click "Create"
## Create a load balancer
- In "Network services", create a load balancer
- Select "HTTP(S) Load Balancing"
- On next screen, keep things kosher
- For Protocol, change to HTTPS
- In IP address, click "CREATE IP ADDRESS", then reserve one for yourself
- In Certificate, create a new one, choose Google-managed, then fill in the domain of your tracker
- For the backend, select that stream collector instance group you've got, then put in 8080 port number
- Scroll down, select the Health check you've created
- Once done, in your tracker domain DNS configuration, add an "A" record that points to the static IP you reserved

# Set up them VMs