---
layout: page
title: Snowplow on GCP
---

[Snowplow](https://snowplow.io/) is a highly customisable behavioural data platform, and blessed be that company since their code is open-source (amen).

For some reason, the only guide for implementing Snowplow on Google Cloud Platform [was written in 2019](https://www.simoahava.com/analytics/install-snowplow-on-the-google-cloud-platform/), so I think it's time for an update. Most of the content below was shamefully lifted directly from the aforementioned article.

# Pre-requisites
You should have your GCP account with billing enabled.

# Enable the required services
Go ahead and switch on:
- Compute Engine API
- Cloud Pub/Sub API

# Install Google Cloud CLI
[It](https://cloud.google.com/sdk/docs/install) makes interacting with GCP easier.

# Service account
If you've used Compute Engine before, your should already a powerful service account set up for you. Otherwise, you can quickly set one up for yourself.