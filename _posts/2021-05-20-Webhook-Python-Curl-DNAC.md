---
title: "Quick and dirty way of testing DNAC webhook with Python"
layout: single
tags:
  - DNAC
  - Python
  - Flask
  - Curl
  - Webhook
toc: true
toc_label: "Outline"
---

Just playing around with Python Flask. Using it as "quick and dirty" way of testing Webhooks.

Webhooks is a way to send notification from a server like DNAC. On the DNAC, you can subscribe to many different types
of notifications and which external destination server to send these notifications to. 

Flask will act as a webhook server, supporting HTTPS and authentication based webhooks. It is just a web server that allows
other servers to post notifications to.
We will use curl command to test fire a webhook subscription at the server, and python code using the request.post to do the same.

I modified the original source [GitHub cisco-en-programmability](https://github.com/cisco-en-programmability/dnacenter_webhook_receiver)
Enabled authentication and allow it to be reachable from the external IP address of an Ubuntu VM (Ubuntu 20.04.2 LTS)

config.py - Change IP address 10.66.69.22 to match where you are running the Flask Python code. WEBHOOK_URL is only used by python code to emulate the notification transmitter **test_webhook.py**
```python
WEBHOOK_URL = 'https://10.66.69.22:5443/webhook'  # test with Flask receiver 
WEBHOOK_USERNAME = 'username'
WEBHOOK_PASSWORD = 'password'
```

You can also used the curl command to emulate the notification transmitter.
```bash
curl --insecure --user "username:password" --header "Content-Type: application/json" --request POST --data '{"Testing_Key":"Testing_Value"}' https://10.66.69.22:5443/webhook
```

***
### Firing a test from **DNAC**

```bash
cisco@ubuntu2:~/Python/webhook$ python3 flask_rx.py
 * Serving Flask app 'flask_rx' (lazy loading)
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
Webhook Received
Payload:
{
    "version" : "1.0.0" ,
    "instanceId" : "1fbf834f-b1ca-42cc-a845-67ce410bea0b" ,
    "eventId" : "NETWORK-DEVICES-3-210" ,
    "namespace" : "ASSURANCE" ,
    "name" : "Interface Flapping On Network Device" ,
    "description" : "A port interface is flapping on a switch" ,
    "type" : "NETWORK" ,
    "category" : "WARN" ,
    "domain" : "Know Your Network" ,
    "subDomain" : "Devices" ,
    "severity" : 3 ,
    "source" : "EXTERNAL" ,
    "timestamp" : 1621494721901 ,
    "details" : {
        "Type" : "" ,
        "Assurance Issue Details" : "Switch  Interface  is flapping" ,
        "Assurance Issue Priority" : "" ,
        "Device" : "" ,
        "Assurance Issue Name" : "Interface  is Flapping on Network Device " ,
        "Assurance Issue Category" : "" ,
        "Assurance Issue Status" : ""
    } ,
    "ciscoDnaEventLink" : "https://&lt;DNAC_IP_ADDRESS&gt;/dna/assurance/issueDetails?issueId=" ,
    "note" : "To programmatically get more info see here - https://<ip-address>/dna/platform/app/consumer-portal/developer-toolkit/apis?apiId=8684-39bb-4e89-a6e4" ,
    "context" : "EXTERNAL" ,
    "userId" : null ,
    "i18n" : null ,
    "eventHierarchy" : null ,
    "message" : null ,
    "messageParams" : null ,
    "parentInstanceId" : null ,
    "network" : null
}
10.66.50.20 - - [20/May/2021 17:12:02] "POST /webhook HTTP/1.1" 202 -
```

![Destination](/assets/images/2021-05-20_destination.jpg)

![Webhook](/assets/images/2021-05-20_webhook.jpg)

![Webhook](/assets/images/2021-05-20_tryit.jpg)