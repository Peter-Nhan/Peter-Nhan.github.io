---
title: "Cisco Live WLC Web Monitoring App"
layout: single
tags:
  - WLC
  - Python
  - Flask
  - netmiko
  - Gunicorn
  - NGINX
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: Exploring the power of Python Flask, we will use Python Flask to act as a Webhook Receive and we will test firing webhooks notification at it via curl, python code and Cisco DNAC. Source Code available on Github.
categories: posts
sitemap: true
pkeywords: Python, Automation, Cisco, Devnet, Devops, Flask, nginx, Cisco, WLC
---
During the recent Cisco Live Melbourne 2023. I was involved in the Cisco NOC (Network Operations) teams. We were tasked to bring our own (bump in) Cisco Networking infrastructure to the MCEC (Melbourne Convention and Exhibition Centre).

We needed a tool to help track AP deployment from our phones, rather than connecting from our laptop to the WLC to check progress and perform health checks.

In this blog, I will discuss the tool I had built to help us track Access Points deployment, and monitor the health of Wireless LAN controller. I wanted the ability to monitor things from our phones with data presented on a web pages that would auto-reload. 

[![](/assets/images/2023-11-11_APs.jpg)](/assets/images/2023-11-11_APs.jpg)

***
### Items used
* Python - was the work horse, I used it extract data from WLC (netmiko), then parse the data into list of dictionary - I made a textfsm version (probably next blog)
* Flask Python class - Use to create our WSGI (Web Server Gateway Interface) application, which renders a dynamic web page - using static web page templates and the data extracted from the WLC.
* Gunicorn - Flask should not be used alone in production environment since is a web framework and not a web server. Gunicorn takes the WSGI Application and translates HTTP requests into something Python can understand.
* NGINX -  public handler (reverse proxy) for incoming requests and scales to thousands of simultaneous connections.

And then we nicely package it all together with two docker containers:
* Gunicorn container - has Gunicorn with the Flask App. Exposes TCP/8081 
* NGINX container - providing reverse proxy for the Gunicorn container. Exposes TCP/8088 but maps it to port 80 internally in the container.


[![](/assets/images/2023-11-11-Docker-HighLevel.png)](/assets/images/2023-11-11-Docker-HighLevel.png)

Most of the work is done within the Flask WSGI App by establishing an SSH session to the WLC using Netmiko Library. Based on a web based query it would proceed to collect and parsing the following commands:
* show ap summary
* show ap summary sort descending client-count
* show ap summary sort descending data-usage
* show wlan all \| include Network Name \| Number of Active Clients

Please Note: Image below has the AP name, IP address, and Mac address pixelated.

[![](/assets/images/2023-11-11-AP-Summary.jpg)](/assets/images/2023-11-11-AP-Summary.jpg)


Before we begin to break down the code (app.py)- you can grab a copy from  [GitHub - Flask webhook receiver](https://github.com/Peter-Nhan/Flask_webhook_receiver){: .btn .btn--primary} <br>

***
### Code break down
> Analysis of app.py - Flask WSGI App

Python file *app.py* has some hardcoded username and password not best practice - this post was more about demonstrating functionality. These credentials are used by the netmiko to connect to the WLC.

The function grab_cli_output collects show commands from the WLC.

{% highlight python linenos %}
from flask import Flask, render_template, send_from_directory
from netmiko import ConnectHandler
import os

app = Flask(__name__)

def grab_cli_output(cli):
    wlc = {
        'device_type': 'cisco_wlc',
        'ip': '192.168.1.1',
        'username': 'admin',
        'password': 'cisco123',
        'secret': 'cisco123',
    }
    # Connect to the WLC
    net_connect = ConnectHandler(**wlc)
    net_connect.enable()
    output = net_connect.send_command(cli)
    # Disconnect from the WLC
    net_connect.disconnect()

    return output
{% endhighlight %}


A number of flask route has been created. This determines how the Flask WSGI App should treat the incoming request.

| Route | Command | Description |
| :--- | :--- | :--- |
| "/" | | Index of all available commands. |
| "/ap_sum" | show ap summary | show total amount AP and their status. | 
| "/ap_sum_client" |  show ap summary sort descending client-count | show AP with most client |
| "/ap_sum_data" | show ap summary sort descending data-usage | show AP with most data usage |

Once the Flask WSGI APP see a request for a route it will trigger a called particular function. <br>
For example if it receives a request for http://x.x.x.x:8081/ap_sum it will be routed to line 6 below. The App will collect and parse the "show ap summary" from the configured WLC, it will then pass the parse values to the template call 

{% highlight python linenos %}
@app.route('/')
def index():
    return render_template('index.html')

# show ap summary
@app.route('/ap_sum')
def show_ap_summary():
    output = grab_cli_output('show ap summary')
    # Split the command output into lines
    lines = output.splitlines()
    # Grab total APs
    total_ap = lines[0]

    # Remove all lines to the actual AP listing
    # Staging-9800-CL#show ap summary
    # Number of APs: 1

    # CC = Country Code
    # RD = Regulatory Domain

    # AP Name                          Slots AP Model             Ethernet MAC   Radio MAC      CC   RD   IP Address                                State        Location
    # ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
    # CW9166_LOANER                    3     CW9166I-Z            6849.9263.a160 ac2a.a1a6.90c0 AU   -Z   10.66.128.225                             Registered   default location
    lines = lines[7:]

    # Parse the AP details
    ap_details = []
    for line in lines:
        ap_info = line.split()
        ap_details.append({
            'ap_name': ap_info[0],
            'slots': ap_info[1],
            'ap_model': ap_info[2],
            'ether_mac': ap_info[3],
            'radio_mac': ap_info[4],
            'country_code': ap_info[5],
            'radio_domain': ap_info[6],
            'ip_address': ap_info[7],
            'state': ap_info[8],
            'location': ap_info[9]
        })

    # Render the template with the AP details
    return render_template('ap_summary.html', ap_details=ap_details, total_ap=total_ap)
{% endhighlight %}

### Template break down
The template uses a combination of CSS boostrap template and variable passed in from our app.py

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>AP Summary</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
</head>
<body>
    <div class="container-md">
    <h1>show ap summary</h1>
    <h2 class="text-end">{{ total_ap }}</h2>
    <table class="table table-hover">
        <tr>
            <thead class="table-light">
            <th>AP Name</th>
            <th>Ethernet MAC Address</th>
            <th>Radio MAC Address</th>
            <th>IP Address</th>
            <th>State</th>
        </tr>
    <tbody class="table-group-divider">
        {% for ap in ap_details %}
        <tr>
            <td>{{ ap.ap_name }}</td>
            <td>{{ ap.ether_mac }}</td>
            <td>{{ ap.radio_mac }}</td>
            <td>{{ ap.ip_address }}</td>
            <td {% if ap.state == 'Registered' %} class="table-success" {% endif %}> {{ ap.state }} </td>
        </tr>
        {% endfor %}
    </table>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
</div>
</body>
</html>
{% endraw %}
{% endhighlight %}

### Docker and config files break down


Directory structure used:
```bash
.
├── app.py
├── dockerfile
├── gunicorn_config.py
├── requirements.txt
├── static
│   ├── CiscoLive-text.png
│   └── favicon.ico
└── templates
    ├── ap_summary_client_count.html
    ├── ap_summary_data_usage.html
    ├── ap_summary.html
    └── index.html
```
***

### Summary
Flask is pretty powerful, with just a few lines of code you can have a fully functioning web server.

**Coming Up Next:** We will further explore ways of customising flask using template to get **Maximum Impact** with very little efforts. And we will continue the same theme of webhook but change the python code to show the content of the log file via the same Flask web service. 

Please reach out if you have any questions or comments.<br>
<i class="fas fa-ghost fa-2x fa-spin"></i>
