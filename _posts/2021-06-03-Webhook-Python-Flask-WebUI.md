---
title: "Python Flask Webhook Receiver with WebUI"
layout: single
tags:
  - DNAC
  - Python
  - Flask
  - Curl
  - Webhook
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: We build on top of the Python Flask Webhook Receiver. Add a WebUI to view the log.
categories: posts
sitemap: true
published: false
pkeywords: Python, Automation, Cisco, Devnet, Flask, DNAC, webhook
---
Taking the Python Flask Webhook Receiver to the next level, but still exploring the power of Flask. We are using Flask to display the content of the json file, which we used to store all the webhook we received.

### Quick demo

[![](/assets/images/2021-06-03_Auto_Refresh_animated.gif)](/assets/images/2021-06-03_Auto_Refresh_animated.gif)

```bash
$ python3 flask_rx_web_view.py
 * Serving Flask app 'flask_rx_web_view' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on https://10.66.69.22:5443/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 768-531-813
10.66.254.152 - - [03/Jun/2021 23:34:12] "GET /log HTTP/1.1" 200 -
10.66.254.152 - - [03/Jun/2021 23:34:22] "GET /log HTTP/1.1" 200 -
10.66.254.152 - - [03/Jun/2021 23:34:32] "GET /log HTTP/1.1" 200 -
Webhook Received
Payload:
{
    "TESTING Via CURL": "Can you see this?"
}
10.66.69.21 - - [03/Jun/2021 23:34:36] "POST /webhook HTTP/1.1" 202 -
10.66.254.152 - - [03/Jun/2021 23:34:42] "GET /log HTTP/1.1" 200 -
```

***
### Code break down

