---
title: "Running Ollama locally on Android device"
layout: single
tags:
  - Android
  - Ollama
  - llm
  - Ollama app
  - samsung
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: This is just the beginning. As mobile hardware continues to advance, and as open-source projects like Ollama mature and optimize for mobile architectures, the idea of having powerful, locally-run AI models in our pockets will become increasingly commonplace.
categories: posts
sitemap: true
pkeywords: Android, Ollama, llm, python, Ollama app, samsung, apk
---

**The Future is Local, and it's Mobile**

This is just the beginning. As mobile hardware continues to advance, and as open-source projects like Ollama mature and optimize for mobile architectures, the idea of having powerful, locally-run AI models in our pockets will become increasingly commonplace.

So, if you've got a reasonably powerful Android phone lying around, and a hankering to get your hands dirty with some cutting-edge AI, I highly recommend exploring both the community "Ollama App" projects and the Termux approach. Dive in, download a small model, and prepare to be amazed at what your phone can do.

**The implications are pretty exciting:**

* Hyper-Personalized AI: Imagine an AI assistant that truly understands your context, your preferences, without ever needing to upload your life to the cloud.

* Creative Tool on the Go: Need a quick brainstorming session? A snippet of code? A poem? Your phone can now be your instant creative partner.
    
* Education and Experimentation: A fantastic way to learn about LLMs, experiment with different models, and understand their capabilities firsthand, without incurring cloud compute costs.

* Accessibility: Opening up LLMs to a wider audience, regardless of their internet access or budget for cloud services.

**Caveats (Because Nothing is Perfect)**

Of course, this is still bleeding edge. Here are a few things to keep in mind:

* Performance Varies: Your mileage will heavily vary depending on your Android device's hardware. Newer, more powerful phones with ample RAM and good chipsets will perform significantly better. 

* Model Size Matters: Stick to smaller models for the better experience. The bigger models will simply eat up too much RAM/storage and be agonizingly slow.

* Battery Life: Running an LLM is computationally intensive. Expect a hit to your battery life.

* Installation & Maintenance: "Ollama Apps" side-loading the APK from GitHub is required, while Termux is available in Google Play Store.

***
### Components used
* **Termux** allows you to run a Linux environment on your Android device. This means you can download and run the official "Ollama server" binaries locally on the Android, giving you the same level of control you'd have on a desktop Linux machine. Termux can be installedfrom Google Play Store.
* **"Ollama App"** allows you to have a user friendly front end to interact with the "Ollama Server" running locally on the Android device. Install the latest "Ollama-App" APK from [Github](https://github.com/JHubi1/Ollama-app/releases)
* **LLM** from [Ollama Library](https://www.Ollama.com/library). Use the library as a reference to determine size and the name to be used.
* **Android device** - Samsung S24 Ultra 12 GB memory and 256 GB storage.

[![](/assets/images/2025-06-01-Android.png)](/assets/images/2025-06-01-Android.png)

***
### Setting up Ollama Server within the Termux App.
Firstly, install "Termux" application from the Google Play Store, once installed open up the application.
Then follow the steps below to install the ollama server.

The following Termux Command line are needed to setup Ollama server.
```
termux-setup-storage
pkg update && pkg upgrade -y
pkg install ollama
```

> All LLM downloaded will be stored with the Termux App - so if Termux App is un-installed or the storage for the application is clear via "clear data" option, then the LLM will be deleted as well.

[![](/assets/images/2025-06-01-Termux-settings.jpeg){: width="250" }](/assets/images/2025-06-01-Termux-settings.jpeg)
[![](/assets/images/2025-06-01-Termux-settings-clear_data.jpeg){: width="250" }](/assets/images/2025-06-01-Termux-settings-clear_data.jpeg)


> The ollama binary is kept in bin folder /data/data/com.termux/files/usr/bin/


### Ollama Server commands
The following are the available Ollama command options

```
$ ollama
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  stop        Stop a running model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information

Use "ollama [command] --help" for more information about a command.
```

Use the command below to start the ollama server, the server will open port 11434, so that "Ollama App" can communicate with it. Alternatively, use 'ollama run' command to run the model inside Termux cli.
We will take the "ollama App" approach since it is easier to use, but the ollama command option is available if you want to stay hardcore!

Start the ollama server with:
```
ollama serve &
```
The following options are only available after the server has started.
* Example: pull(Download) and run deepseek-r1:1.5b
```
ollama run deepseek-r1:1.5b
```

* Example: pull(Download) deepseek-r1:1.5b
```
ollama pull deepseek-r1:1.5b
```

* Example: List what models are available
```
ollama list
```

Once the ollama server has started, the server will open port 11434, so that "Ollama App" can communicate with it.

### Ollama App
Download the "Ollama App" from the github link above and install the APK.
Configure the Ollama App, bychanging the "Settings" host to point to ourselves (localhost) on port 11434.
```
http://localhost:11434
```
This will allow us to interact with Ollama Server through the Ollama App interface.

[![](/assets/images/2025-06-01-ollama-setting-1.jpeg){: width="250" }](/assets/images/2025-06-01-ollama-setting-1.jpeg)
[![](/assets/images/2025-06-01-ollama-setting-2.jpeg){: width="250" }](/assets/images/2025-06-01-ollama-setting-2.jpeg){: width="250" }

The ollama app allows the user to interact with the selected model and allows for new LLM to be downloaded from the Ollama library.

[![](/assets/images/2025-06-01-Termux-memory.jpg){: width="250" }](/assets/images/2025-06-01-Termux-memory.jpg){: width="250" }
[![](/assets/images/2025-06-01-ollama-App.gif){: width="250" }](/assets/images/2025-06-01-ollama-App.gif){: width="250" }

With Termux running and nothing else, the phone has about 4.5 Gigs of RAM free. I have tested with some smaller LLM from Ollama Library. There is some latency in the reply, but still pretty good for a phone.

LLM's I have tested with:
* deepseek-r1 - 1.5b
* gemma3 - 1b
* qwen3 - 1.7b


### Summary
So there you have it. The days of needing a server farm or a monstrous desktop rig to play with large language models are fading fast. We're now firmly in an era where the power of local AI is accessible right in the palm of your hand.

So, go on. Dive in. Your Android phone is no longer just a communication device; it's now a bona fide AI playground. The revolution isn't coming; it's already running locally on your device.

Please reach out if you have any questions or comments.<br>
<i class="fa-solid fa-envelope"></i>
<i class="fa-solid fa-heart fa-beat" style="--fa-beat-scale: 2.0;"></i>