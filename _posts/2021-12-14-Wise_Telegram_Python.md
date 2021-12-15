---
title: "Automate Transferwise with Python and Telegram"
layout: single
tags:
  - Python
  - wise
  - TransferWise
  - Telegram
  - bot
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: Automate Transferwise currency check with Python and Telegram.
categories: posts
sitemap: true
published: true
pkeywords: Python, Automation, Wise, TransferWise, Telegram, Bot
---
I have been using Wise or Transferwise (old name) for a while now. For those who do not know, Wise allows you to do currency exchange and transfer money around the world at a low cost. I am not sponsored by them, happy to be ... hint hint.. :-)

Jokes aside, when I found out there was API available to interact with the Wise platform I was quite excited. One of the biggest pain point for me in using the platform or the App, is that it does not let you set a target exchange rate to trigger the transfer. So you had to manually check the App or webpage constantly.

For this blog, I want to focus on the basic functionality. I want to be informed when the AUD to USD exchange rate hit my target. So that I can manually trigger the conversion. I want Telegram app to send me an alert when the target has reached.
I also want to set my target through the Telegram App and have python check the exchange rate every 10 minutes (can be modified) and have the ability to turn off the alert via the Telegram App as well.
There is a Wise API to trigger the currency conversion for you but I am not so brave yet. 
<i class="far fa-grin-beam-sweat"></i>

***
### My Setup
Currently, I have the python3 code running on my Raspberry Pi at home. The Python3 code leverages Telegram API to monitor the bot for activity and to send messages to the bot. The code also uses the Wise API to check for currency exchange rates.

Outline of the Telegram app commands implemented:

**/tw_on** <AUD_USD target> - check exchange rate every 600 seconds - if checked exchange rate is better or equal to the target exchange rate then Send Message

**/tw_off** - turn off currency checks

**/tw_now** - currency checks now

Animated gif has been edited to be fast forward 10 mins ahead, from "/tw_on .." command to show Python responding with a currency check, since the currency price is better than targeted one.

[![](/assets/images/2021-12-14-Wise.gif)](/assets/images/2021-12-14-Wise.gif)

***
### Prerequisites
Before we proceed, there are some pre-requisites:
* [Register for a Free Wise Sandbox account](https://sandbox.transferwise.tech/register/){:target="_blank"} and then grab the API Key. We need this API Key to talk to the Wise sandbox. This way we can use the Wise API that is available. Wise API reference link below. Please do not share the API key with anyone. 
[![](/assets/images/2021-12-14-WiseAPIKey.png)](/assets/images/2021-12-14-WiseAPIKey.png)

* Create a bot in Telegram to get the Token for the bot. Start off by adding BotFather to your telegram and follow the steps in the screenshots below to create a bot of your own. Use you own "Name for your bot" and username for your bot. Grab the API token from BotFather - do not share with anyone

[![](/assets/images/2021-12-14-Telegram1.png)](/assets/images/2021-12-14-Telegram1.png)

[![](/assets/images/2021-12-14-Telegram2.png)](/assets/images/2021-12-14-Telegram2.png)

Please refer to the Wise API reference guide for more details and for more other APi calls.
[Wise API Reference - Exchange Rates](https://api-docs.transferwise.com/partners#exchange-rates-list){:target="_blank"}

    https://api.sandbox.transferwise.tech/v1/rates?source=USD&target=AUD

To follow along, the files are available here. [Github](https://github.com/Peter-Nhan/Wise_Telegram_Python){: .btn .btn--primary}

***
### Code break down
> Analysis of wise-bot.py  - Telegram Wise Python Bot

Python file *wise-bot.py* needs the two API keys to be updated before we can proceed, see above on how to get them:
- Line 8 - Needs the TransferWise Auth Token (API Key) 
- Line 10 - Needs the Telegram Bot Token (API Token)
- Line 12 - check currency every 600 seconds - can be modified to what ever you like
- Line 23-36 - function describes what functions is available as well as print some extra bits of informations
- Line 38-48 - function that check the exchange rate and is schedule to be re-occurring
- Line 50-71 - function that takes the exchange rate from the user and kicks off the tw_check function for re-occurring
- Line 73-79 - function cancels the re-occurring currency checks
- Line 81-86 - function that returns the currency now
- Line 106-110 - the mapping of keywords that Python is listening on from Telegram and the function that will be triggered if there is a match.

{% highlight python linenos %}
#!/usr/local/bin/python3
import logging
import requests
from telegram import Update, message
from telegram.ext import Updater, CommandHandler, CallbackContext

# Get Transferwise Auth token from the sandbox account
TransferWise_Auth_token = ""
# Get Telegram BOT token from the BotFather in Telegram App
Telegram_Bot_token = ""
AudUsdExchange = 1.00
tw_check_interval = 600
tw_token_headers = {'content-type': 'application/json', 'accept':'application/json', 'Authorization' : 'Bearer '+ TransferWise_Auth_token}
tw_url = "https://api.sandbox.transferwise.tech/v1/rates?source=USD&target=AUD"

# Enable logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

logger = logging.getLogger(__name__)

# Define a few command handlers. These usually take the two arguments update and
# context. Error handlers also receive the raised TelegramError object in error.
def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text('*_Wise Bot_*', parse_mode='MarkdownV2')
    update.message.reply_text("Chat_id = {} ".format(update.message.chat_id))
    update.message.reply_text('Use /tw_on <AUD_USD target> - check exchange rate every '+str(tw_check_interval)+' seconds - if checked exchange rate is better or equal to the target exchange rate then Send Message')
    update.message.reply_text('Use /tw_off - turn off currency checks')
    update.message.reply_text('Use /tw_now - currency checks now')
    print("===> /start message ")

    if(update.message.chat.id < 0):
        print("Message Chat ID - Group: {}".format(update.message.chat.id))
        print("Message Chat Title: {}".format(update.message.chat.title))
    else:
        print("Message Chat ID: {}".format(update.message.chat.id))
        print("Message Chat Individual Name: {}".format(update.message.chat.first_name))

def tw_check(context: CallbackContext) -> None:
    """Send the tw_check message."""
    print("===> TransferWise - checking exchange rate now")
    global AudUsdExchange
    job = context.job
    response = requests.get(tw_url, headers=tw_token_headers)
    if (AudUsdExchange >= 1/response.json()[0]['rate']):
        context.bot.send_message(job.context, text='Current Rate: {} - Watch Rate: {}'.format(1/response.json()[0]['rate'],AudUsdExchange))
        print("===> TransferWise = Good exchange rates - Current Rate: {} - Watch Rate: {}".format(1/response.json()[0]['rate'],AudUsdExchange))
    else:
        print("===> TransferWise = Bad exchange rates - Current Rate: {} - Watch Rate: {}".format(1/response.json()[0]['rate'],AudUsdExchange))

def tw_on(update: Update, context: CallbackContext) -> None:
    """Add a job to the queue."""
    print("===> TransferWise - turn ON regular checks")
    chat_id = update.message.chat_id
    global AudUsdExchange
    try:
        # args[0] should contain the time for the timer in seconds
        AudUsdExchange = float(context.args[0])
        if AudUsdExchange < 0:
            update.message.reply_text('Negative Exchange rates?')
            return

        job_removed = remove_job_if_exists(str(chat_id), context)
        context.job_queue.run_repeating(tw_check, tw_check_interval, context=chat_id, name=str(chat_id))

        text = 'AUD_USD target check set - ' + str(AudUsdExchange)
        if job_removed:
            text += ' -> Old currency check was removed.'
        update.message.reply_text(text)

    except (IndexError, ValueError):
        update.message.reply_text('Usage: /tw_on <AUD_USD target>')

def tw_off(update: Update, context: CallbackContext) -> None:
    """Remove the job if the user changed their mind."""
    print("===> TransferWise - turn OFF regular checks")
    chat_id = update.message.chat_id
    job_removed = remove_job_if_exists(str(chat_id), context)
    text = 'Currency check cancelled!' if job_removed else 'You have no active timer.'
    update.message.reply_text(text)

def tw_now(update: Update, context: CallbackContext) -> None:
    """Remove the job if the user changed their mind."""
    print("===> TransferWise - check now")
    chat_id = update.message.chat_id
    response = requests.get(tw_url, headers=tw_token_headers)
    update.message.reply_text("Currency now - {}".format(1/response.json()[0]['rate']))

def remove_job_if_exists(name: str, context: CallbackContext) -> bool:
    """Remove job with given name. Returns whether job was removed."""
    current_jobs = context.job_queue.get_jobs_by_name(name)
    if not current_jobs:
        return False
    for job in current_jobs:
        job.schedule_removal()
    return True

def main() -> None:
    """Run bot."""
    # Create the Updater and pass it your bot's token.
    updater = Updater(Telegram_Bot_token)

    # Get the dispatcher to register handlers
    dispatcher = updater.dispatcher

    # on different commands - answer in Telegram
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("help", start))
    dispatcher.add_handler(CommandHandler("tw_on", tw_on))
    dispatcher.add_handler(CommandHandler("tw_off", tw_off))
    dispatcher.add_handler(CommandHandler("tw_now", tw_now))

    # Start the Bot
    updater.start_polling()

    # Block until you press Ctrl-C or the process receives SIGINT, SIGTERM or
    # SIGABRT. This should be used most of the time, since start_polling() is
    # non-blocking and will stop the bot gracefully.
    updater.idle()

if __name__ == '__main__':
    main()
{% endhighlight %}

***
### Summary
The interactions Python and Telegram is a convenient way to remotely trigger functions. Hopefully it has provide some ideas of other projects you code do.
As always, please reach out if you have any questions or comments or suggestions.<br>
<i class="far fa-comment-dots fa-2x"></i>
