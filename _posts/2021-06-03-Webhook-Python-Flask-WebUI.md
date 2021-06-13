---
title: "Python Flask Webhook Receiver with WebUI"
layout: single
tags:
  - DNAC
  - Python
  - Flask
  - Curl
  - Webhook
  - bootstrap
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: We build on top of the Python Flask Webhook Receiver. Add a WebUI to view the log. Most of the HTML code would be bootstrapped. And we will talk about flask and Jinja2 relationship
categories: posts
sitemap: true
published: true
pkeywords: Python, Automation, Cisco, Devnet, Flask, DNAC, webhook, bootstrap, jinja2
---
Taking the Python Flask Webhook Receiver to the next level, but still exploring the power of Flask. We are using Flask to display the content of the json file, which we used to store all the webhook we received. We will use and discuss a technique of building HTML called Bootstrap.

To follow along, the files are available here. [Github](https://github.com/Peter-Nhan/Flask_webhook_receiver).

{: .notice--info} 
<i class="fa fa-exclamation-triangle fa-2x" style="color:yellow"></i> <b>Important:</b><br>
If you are playing along - remember to 'pip3 install requirements.txt'. This will install the required libraries used by the python script.

### Quick demo

After kicking off the flask_rx_web_view.py, I can browse to https://10.66.69.22:5443/log (ignore the security pop up - we do not have proper certificate in place - "Accept the Risk and Continue"). The webpage will display the content of the log and will refreshed every 10 seconds. I have also test fire a curl webhook at the webhook receiver, and you can see it update after the 10 seconds. 

{: .notice--info} 
<i class="fa fa-exclamation-triangle fa-2x" style="color:yellow"></i> <b>Important:</b><br>
Your IP address of the device you are running the python code maybe different to mine. The IP address where I ran the 
'python3 flask_rx_web_view.py' is 10.66.69.22.

The top of the web page has a navigation bar, that has options to allow you to manually refreshed the page and a button to download the json file. These were added by "bootstrapping" the HTML in the template folder. More details below.

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
There are two parts to this Code Break Down section:

**First part is the HTML** - bootstrap.html which lives in the templates folder, is in reality a Jinja2 HTML template. 
```bash
├── all_webhooks_detailed.json
├── config.py
├── flask_rx.py
├── flask_rx_web_view.py
├── requirements.txt
├── templates
│   └── bootstrap.html
├── test_webhook.py
└── userpass_base64.py
```
Flask allows the HTML template to be render with variables produced from python code. Flask leverages Jinja2 as its template engine to do this.

The heavily lifting in the HTML template was done by the folks at:
[https://getbootstrap.com/docs/5.0/getting-started/introduction/](https://getbootstrap.com/docs/5.0/getting-started/introduction/)

They have lots of examples on how to leverage there code - [https://getbootstrap.com/docs/5.0/examples/](https://getbootstrap.com/docs/5.0/examples/)

They have made it so easy to get a decent website up and running. They provide all the CSS and Javascript, and design templates for you to use.

I used their example Navbar to build what you see above demo - [https://getbootstrap.com/docs/5.0/components/navbar/](https://getbootstrap.com/docs/5.0/components/navbar/)

Buttons looked pretty cool too - https://getbootstrap.com/docs/4.0/components/buttons/

I have already heavily commented the included HTML template(bootstrap.html). Most of it, is about making the webpage look decent. But I will go through the core part of the HTML template that produces the dynamic content you see.

The HTML code you see below is a snippet from the bootstrap.html. 

This section of bootstrap.html makes the web page refresh every 10 seconds.
```html
<!-- Automatically refresh web page every 10 seconds  -->    
    <meta http-equiv="refresh" content="10">
```
This section of bootstrap.html creates two links:
* "Refresh Page" link manually refreshes the current web page
* "Download all_webhooks_detailed.json" button to allow you to download the file - **Note** the variable that is surrounded double curly brackets (filenamez_var), thiswas  passed into the HTML code via the python call (see next section)

```html
{% raw %}
<!-- Create a link on web page call "Refresh Page" to reload the page manually -->
          <li class="nav-item active">
            <a class="nav-link" href="/log">Refresh Page
            </a>
          </li>
<!-- Create a button link on web page to download the json file -->          
          <li class="nav-item active">
            <a button type="button" href="/download" class="btn btn-success">Download {{ filename_var }}</button>
            </a>  
          </li>
{% endraw %}
```

Notice the doubly curly brackets are the two Jinja2 variable:
* filename_var - variable has the filename of the log.
* content_var - content of the log file is store here.
These content are passed in via the flask in the python code.

```html
{% raw %}
    <main role="main" class="container">
      <div class="starter-template">
        <div class="alert alert-success" role="alert">
<!-- Variable filename_var pass in via python code - variable is like jinja2  -->   
          Refresh page - to get the latest changes in {{ filename_var }}
        </div>
<!-- Python variable content_of_file values are added below -->
      <pre>{{ content_var }}</pre>
    </div>
  </main>
{% endraw %}
```



**Second part is the python** code - we create two more route for **/log** and **/download**.  They are trigger when you use https://x.x.x.x:5433/log or https://x.x.x.x:5433/download.

- **https://x.x.x.x:5433/log**
Once triggered it will open a filename *all_webhooks_detailed.json*, and save the content into a variable called *content_of_file*. The flask **render_template** function will take these variable and passed the into the HTML template. And Flask will then render the output of those variables into what you see in the web page.

|Python Variable|HTML Variable|
|---------------|----------------|
| content_of_file, save_webhook_output_file | content_var, filename_var  |


{% highlight python linenos %}
save_webhook_output_file = "all_webhooks_detailed.json"
..
..
# Access the logs via the Web 
@app.route("/log", methods=['GET'])  # create a route for /log, method GET
def log():
    with open(save_webhook_output_file, "r") as f: 
        content_of_file = f.read() 
    return render_template('bootstrap.html', content_var = content_of_file, filename_var = save_webhook_output_file)
{% endhighlight %}

- **https://x.x.x.x:5433/downloads**
Once this call is triggered it will send the file with filename stored in the variable *save_webhook_output_file* to clients web browser.

{% highlight python linenos %}
# Download the logs file
@app.route("/download", methods=['GET'])  # create a route for /download, method GET
def download():
    return send_file(save_webhook_output_file, as_attachment=True)
{% endhighlight %}

***
### Summary
Bootstraping the HTML, means you do not need to know a lot about HTML. Combined that with Flask and you quickly and easily create great looking portal or webpages.

**Coming Up Next:** Going to have a go at dockerising the python code and point out any pitfalls during the process.

As always, please reach out if you have any questions or comments or suggestions.<br>
<i class="fas fa-ghost fa-2x fa-spin"></i>
