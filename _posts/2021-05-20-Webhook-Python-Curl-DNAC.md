---
title: "Python Flask Webhook Receiver"
layout: single
tags:
  - DNAC
  - Python
  - Flask
  - Curl
  - Webhook
toc: true
toc_label: "Outline"
toc_icon: "fas fa-gamepad"
toc_sticky: True
---
Just playing around with Python Flask. Using it as a "quick and dirty" way of testing Webhooks.
Webhooks (Reverse API) is a way to send notification from one application to another application. 

Python Flask will act as a webhook receiver, supporting HTTPS and authentication based webhooks.

Before we begin, some details about the Webhook Flask receiver. I modified the original source [GitHub cisco-en-programmability](https://github.com/cisco-en-programmability/dnacenter_webhook_receiver) and fork the changes here [Flask webhook receiver](https://github.com/Peter-Nhan/Flask_webhook_receiver). I enabled authentication and allow it to be reachable from the external IP address of an Ubuntu VM (Ubuntu 20.04.2 LTS)

> Analysis of flask_rx.py

Python file *flask_rx.py* import value of the username and password from *config.py*. These credentials are used, when you post notification to the webhook receiver.
You can also customise the filename that is used to save all the received webhook notification 'save_webhook_output_file'.

{% highlight python linenos %}
from config import WEBHOOK_USERNAME, WEBHOOK_PASSWORD
save_webhook_output_file = "all_webhooks_detailed.json"

app = Flask(__name__)

app.config['BASIC_AUTH_USERNAME'] = WEBHOOK_USERNAME
app.config['BASIC_AUTH_PASSWORD'] = WEBHOOK_PASSWORD

# If true, then site wide authentication is needed
app.config['BASIC_AUTH_FORCE'] = True

basic_auth = BasicAuth(app)
{% endhighlight %}
A number of flask route has been created - "/" and "/webhook". This determines how the flask web service should treat the incoming request.
* "/" is used to test if the web service is running.
* "/webhook" is used to post the webhook notification - so the full URL would be https://x.x.x.x/webhook
Once the webhook is received we use the json.dumps to print it to the screen and as well as dump it to the file.

{% highlight python linenos %}
@app.route('/')  # create a route for / - just to test server is up.
@basic_auth.required
def index():
    return '<h1>Flask Receiver App is Up!</h1>', 200

@app.route('/webhook', methods=['POST'])  # create a route for /webhook, method POST
@basic_auth.required
def webhook():
    if request.method == 'POST':
        print('Webhook Received')
        request_json = request.json

        # print the received notification
        print('Payload: ')
        # Change from original - remove the need for function to print
        print(json.dumps(request_json,indent=4))

        # save as a file, create new file if not existing, append to existing file
        # full details of each notification to file 'all_webhooks_detailed.json'
        # Change above save_webhook_output_file to a different filename

        with open(save_webhook_output_file, 'a') as filehandle:
            # Change from original - we output to file so that the we page works better with the newlines.
            filehandle.write('%s\n' % json.dumps(request_json,indent=4))
            filehandle.write('= - = - = - = - = - = - = - = - = - = - = - = - = - = - = - = - = - = - \n')

        return 'Webhook notification received', 202
    else:
        return 'POST Method not supported', 405
{% endhighlight %}

We use the app.run to specified the port 5443, HTTPS and to bind Flask to all IP addresses on the Ubuntu VM.
{% highlight python linenos %}
if __name__ == '__main__':
    # HTTPS enable - toggle on eby un-commenting
    app.run(ssl_context='adhoc', host='0.0.0.0', port=5443, debug=True)
{% endhighlight %}

We will use three different method to test the Python Flask Webhook receiver 

1. Curl command to test fire a webhook subscription at the receiver, 
2. Use another Python code - *test_webhook.py* using the request.post fire the request
3. On the Cisco DNAC, you can subscribe to many different types of event notifications and add external destination receiver to send these notifications to. Configure DNAC with the details of the webhook receiver and we can test fire from there.

[![](/assets/images/2021-05-20_Curl_test_webhook.png)](/assets/images/2021-05-20_Curl_test_webhook.png)

[![](/assets/images/2021-05-20_DNAC_Webhook.png)](/assets/images/2021-05-20_DNAC_Webhook.png)

{: .notice--danger}
Updated *config.py* to change IP address 10.66.69.22 to match where you are running the Flask Python code. 
WEBHOOK_URL is only used by python code *test_webhook.py* to emulate the notification transmitter.
Using the WEBHOOK_USERNAME and WEBHOOK_PASSWORD values from *config.py*.

```python
WEBHOOK_URL = 'https://10.66.69.22:5443/webhook'  # test with Flask receiver 
WEBHOOK_USERNAME = 'username'
WEBHOOK_PASSWORD = 'password'
```
***
### Firing a test from *curl* command
The curl command to emulate the notification transmission.
The data is any payload - intent was to test functionality. You can customer the content of the dictionary after "--data"
> Test done from my Mac

```bash
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$ curl --insecure --user "username:password" --header "Content-Type: application/json" --request POST --data '{"Testing_Key":"Testing_Value"}' https://10.66.69.22:5443/webhook
Webhook notification received%
[pnhan@PNHAN-M-1466 ~/Documents/Python/webhook]$
```
> Meanwhile, on the Ubuntu VM it was running *flask_rx.py*. It received the notification from our curl command - Note the payload is displayed.

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
> Meanwhile, on the Ubuntu VM it was running *flask_rx.py*. It received the notification from test_webhook.py - Note the payload is displayed.

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

Add the credential of the webhook. 

[![](/assets/images/2021-05-20_webhook.jpg)](/assets/images/2021-05-20_webhook.jpg)

Note the Authorization headers is a Base64 encoding of username:password
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

Subscribe to a notification and "Try it"
[![](/assets/images/2021-05-20_tryit.jpg)](/assets/images/2021-05-20_tryit.jpg)

> Meanwhile, on the Ubuntu VM it was running *flask_rx.py*. It received the notification from Cisco DNAC - Note the payload is displayed.

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
<i class="fas fa-ghost"></i>