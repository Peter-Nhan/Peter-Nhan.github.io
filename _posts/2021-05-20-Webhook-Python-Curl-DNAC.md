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

Webhooks is a way to send notification from a application server like Cisco DNAC to another application. On the Cisco DNAC, you can subscribe to many different types of notifications and which external destination server to send these notifications to. 

Python Flask will act as a webhook server, supporting HTTPS and authentication based webhooks. It is just a web server that allows other servers to post notifications to. (Reverse API)

We will use three different method of test the Python Flask Webhook server 

1. Curl command to test fire a webhook subscription at the server, 
2. Use another Python code - **test_webhook.py** using the request.post fire the request
3. Configure DNAC with the details of the webhook server and we can test fire from there.

Before we begin, some details about the Webhook Flask server. I modified the original source [GitHub cisco-en-programmability](https://github.com/cisco-en-programmability/dnacenter_webhook_receiver)

Enabled authentication and allow it to be reachable from the external IP address of an Ubuntu VM (Ubuntu 20.04.2 LTS)
I updated **config.py** - Change IP address 10.66.69.22 to match where you are running the Flask Python code. 
WEBHOOK_URL is only used by python code to emulate the notification transmitter **test_webhook.py**

```python
WEBHOOK_URL = 'https://10.66.69.22:5443/webhook'  # test with Flask receiver 
WEBHOOK_USERNAME = 'username'
WEBHOOK_PASSWORD = 'password'
```
***

### Firing a test from *curl* command
The curl command to emulate the notification transmission.
The data is any payload - intent was to test it works.
> Test done from my Mac

```bash
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$ curl --insecure --user "username:password" --header "Content-Type: application/json" --request POST --data '{"Testing_Key":"Testing_Value"}' https://10.66.69.22:5443/webhook
Webhook notification received%
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$
```
> Meanwhile, on the Ubuntu VM it was running *flask_rx.py". It received the notification from our curl command - Note the payload is displayed.

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
    "Testing_Key" : "Testing_Value"
}
10.66.246.93 - - [22/May/2021 21:10:23] "POST /webhook HTTP/1.1" 202 -
```
***
### Firing a test from *test_webhook.py* command
> Test done from my Mac but using the test_webhook.py - note it has some pre-defined payload

```bash
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$ python3 test_webhook.py
Webhook notification status code:  202
Webhook notification response:  Webhook notification received
```

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
    "version" : "" ,
    "instanceId" : "ea6e28c5-b7f2-43a4-9937-def73771c5ef" ,
    "eventId" : "NETWORK-NON-FABRIC_WIRED-1-251" ,
    "namespace" : "ASSURANCE" ,
    "name" : "" ,
    "description" : "" ,
    "type" : "NETWORK" ,
    "category" : "ALERT" ,
    "domain" : "Connectivity" ,
    "subDomain" : "Non-Fabric Wired" ,
    "severity" : 1 ,
    "source" : "ndp" ,
    "timestamp" : 1574457834497 ,
    "tags" : "" ,
    "details" : {
        "Type" : "Network Device" ,
        "Assurance Issue Priority" : "P1" ,
        "Assurance Issue Details" : "Interface GigabitEthernet1/0/3 on the following network device is down: Local Node: PDX-M" ,
        "Device" : "10.93.141.17" ,
        "Assurance Issue Name" : "Interface GigabitEthernet1/0/3 is Down on Network Device 10.93.141.17" ,
        "Assurance Issue Category" : "Connectivity" ,
        "Assurance Issue Status" : "active"
    } ,
    "ciscoDnaEventLink" : "https://10.93.141.35/dna/assurance/issueDetails?issueId=ea6e28c5-b7f2-43a4-9937-def73771c5ef" ,
    "note" : "To programmatically get more info see here - https://<ip-address>/dna/platform/app/consumer-portal/developer-toolkit/apis?apiId=8684-39bb-4e89-a6e4" ,
    "tntId" : "" ,
    "context" : "" ,
    "tenantId" : ""
}
10.66.246.93 - - [22/May/2021 21:17:06] "POST /webhook HTTP/1.1" 202 -
```
***

### Firing a test from *Cisco DNAC*

Add the webhook destination in DNAC
[![](/assets/images/2021-05-20_destination.jpg)](/assets/images/2021-05-20_destination.jpg)

Add the credential of the webhook. Note the Authorization headers is a Base64 encoding of username:password
If you need help with the Base64 encoding - I have included *userpass_base64.py* to help.
```
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$ python3 userpass_base64.py
Enter Username: username
Enter Password: password
username:password
Encoded string: dXNlcm5hbWU6cGFzc3dvcmQ=
Header for Basic Authentication
Authorization Header value: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

[![](/assets/images/2021-05-20_webhook.jpg)](/assets/images/2021-05-20_webhook.jpg)

Subscribe to a notification and "Try it"
[![](/assets/images/2021-05-20_tryit.jpg)](/assets/images/2021-05-20_tryit.jpg)

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
