---
title: "Quick and dirty way of testing webhook with Python and DNAC"
tags:
  - DNAC
  - Python
  - Flask
  - Curl
  - Webhook
toc_label: "Outline"
---

Just playing around with Python Flask. Using it as "quick and dirty" way of testing Webhooks.

Webhooks is a way to send notification from a server like DNAC. On the DNAC, you can subscribe to many different types
of notifications and which external destination server to send these notifications to. 

Flask will act as a webhook server, supporting HTTPS and authentication based webhooks. It is just a web server that allows
other servers to post notifications to.
We will use curl command to test fire a webhook subscription at the server, also a python way using the request.post way.

I modified the original source [GitHub cisco-en-programmability](https://github.com/cisco-en-programmability/dnacenter_webhook_receiver)
Enabled authentication and allow it to be reachable from external IP address of a Ubuntu VM (Ubuntu 20.04.2 LTS)




