---
title: "Running LLM locally with Ollama and Open WebUI"
layout: single
tags:
  - docker
  - ollama
  - llm
  - python
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: To truly understand the hype behind AI, I needed to get my hands dirty and not just read or watch endless video about the topic. Here is a blog write up on running Local LLM with Ollama with Cisco UCS Compute.
categories: posts
sitemap: true
pkeywords: docker, ollama, llm, python, langchain,
---
One of the job perks is having access to great bunch of toys you can play and experiment with.
For me to truly understand the hype behind AI, I needed to get my hands dirty and not just read or watch endless video about the topic.
This is a blog is about running Local LLM with Ollama and Open WebUI, this way you can have your LLM privately and not on the cloud.
This allows you leaverage AI without having to risk your personal details being shared or used by the cloud providers.

Here is a quick outline:
* Details about docker containers for Ollama (Server) and Ollama Web UI (front end).
* Explore the guard rails, and bias.
	* Asking the Meta's LLama LLM how to perform illegal activities on original censored LLM and uncensored LLM
	* Asking the Alibaba's QWEN LLM about controversial topic that are sensitive within China due to strict government censorship.
* Interact with Ollama via Python libraries, and direct interaction within the container.
* Baseline comparison of LLM on how much RAM memory are used for each LLM (since I did not have a GPU - yet) from Microsoft Phi3 (3 Billions parameters) to Qwen2 (72.7 Billions parameters)

***
### Components used
* [Ollama Server](https://ollama.com/) - a platform that make easier to run LLM locally on your compute.
* [Open WebUI](https://github.com/open-webui/open-webui) - a self hosted front end that interacts with APIs that presented by Ollama or OpenAI compatible platforms.
* Linux Server or equivalent device - spin up two docker containers with the Docker-compose YAML file specified below.

***
### Code break down
> Analysis of Docker-Compose.yml file

{% highlight yaml linenos %}
---
services:
  ollama-server:
    image: ollama/ollama:latest
    container_name: ollama-server
    ports:
      - "11434:11434"
    volumes:
      - ./ollama_data:/root/.ollama
    restart: unless-stopped

  ollama-webui:
    image: ghcr.io/ollama-webui/ollama-webui:main
    container_name: ollama-webui
    restart: unless-stopped
    environment:
      - 'OLLAMA_BASE_URL=http://ollama-server:11434'
    volumes:
      - ./webui:/app/backend/data
    ports:
      - "3010:8080"
    extra_hosts:
      - host.docker.internal:host-gateway
{% endhighlight %}

*Line 7* - Ollama Server exposes port 11434 for its API.

*Line 9* - maps a folder on the host ollama_data to the directory inside the container /root/.ollama - this is where all LLM are downloaded to.

*Line 17* - environment variable that tells Web UI which port to connect to on the Ollama Server. Since both docker containers are sitting on the same host we can refer to the ollama container name 'ollama-server' in the URL.

*Line 21* - Connect to the Web UI on port 3010.

*Line 22-23* - Avoids the need for this container to use 'host' network mode.

My directory structure in the folder where docker compose exist.

```
❯ tree -d -L 1
.
├── ollama_data
└── webui

3 directories
```

Issue 'docker compose up -d' from the folder where your docker compose file are, to install and start the containers.
Once both containers are up, you can browse to the ollama webui on port 3010 - example http://x.x.x.x:3010

[![](/assets/images/2024-07-12-Ollama-Container.gif)](/assets/images/2024-07-12-Ollama-Container.gif)
### Interact with Ollama via the Web UI

Once you connected to the Web UI from a browser it will ask you to set up a local account on it.

[![](/assets/images/2024-07-12-SignUp.png)](/assets/images/2024-07-12-SignUp.png)

After which you can go ahead and download the LLM you want to use.

[![](/assets/images/2024-07-12-pull-image.png)](/assets/images/2024-07-12-pull-image.png)

The screenshot below is testing the guard rails the llama3 LLM (Meta) have in place. The guard rails in this case is not to provide information for illegal activities.

[![](/assets/images/2024-07-12-llama3-guardrails.png)](/assets/images/2024-07-12-llama3-guardrails.png)

This next screenshot below is testing the uncensored version of llama3 called dolphin-llama3. You can see in this case the guard rails have been removed.

[![](/assets/images/2024-07-12-dolphin-llama3-guardrails.png)](/assets/images/2024-07-12-dolphin-llama3-guardrails.png)

### Interact with Ollama via CLI in the docker container
Use the docker exec command to start a bash shell inside the container. Once within the container there is a CLI command call 'ollama'
```
# docker exec -it ollama-server bash

root@9001ce6503d1:/# ollama
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command
```

To pull down an LLM and show what LLM models are available locally.
To see all the available LLM visit the [ollama website](https://ollama.com/library).
```
root@9001ce6503d1:/# ollama pull gemma2
pulling manifest
pulling ff1d1fc78170... 100% ▕███████████████████████████████████████ ▏ 5.4 GB/5.4 GB  312 KB/s     26s

root@9001ce6503d1:/# ollama list
NAME                    ID              SIZE    MODIFIED
qwen2:72b               14066dfa503f    41 GB   8 hours ago
phi3:latest             d184c916657e    2.2 GB  10 hours ago
mistral:latest          2ae6f6dd7a3d    4.1 GB  10 hours ago
gemma2:latest           ff02c3702f32    5.4 GB  11 hours ago
dolphin-llama3:latest   613f068e29f8    4.7 GB  2 days ago
llama3:latest           365c0bd3c000    4.7 GB  2 days ago
```

The following command runs the selected LLM and issues a query to the LLM (QWEN2 - made by Alibaba Cloud). And keeps the LLM loaded for 2 minutes by default it is 5 minutes.

```
root@9001ce6503d1:/# ollama run qwen2:72b --keepalive 2m
>>> what happen at the protest in tiananmen square in 1989?
The Tiananmen Square protests of 1989 were a series of demonstrations and civil movements that took place in Beijing's Tiananmen Square from April to June. The events
culminated in a violent confrontation between the protesters and government forces on June 4, 1989.

Here is a summary of what happened during these protests:

1. **Funeral Protests:** The initial demonstrations began as a memorial for Hu Yaobang, a former General Secretary of the Chinese Communist Party who had been purged
from power due to his calls for political reform and greater openness in government. His sudden death on April 15 sparked protests among students and intellectuals.

2. **Expanding Demands:** The demonstrations grew beyond mourning Hu's death to encompass broader demands such as freedom of speech, the press, and association; an end
to official corruption; fair competitive examinations for university admissions; and other democratic reforms.

3. **Growth of Protests:** These demonstrations soon attracted a wide range of participants from across Chinese society, including workers, intellectuals, and even
some government officials who sympathized with the protesters' demands. At its peak, it's estimated that hundreds of thousands to over one million people were gathered
in Tiananmen Square.

4. **Government Response:** The Chinese government initially took a cautious approach but as the protests continued, hardliners within the party gained influence. On
May 19, Premier Li Peng declared martial law, and troops began to be deployed around Beijing. However, some military units refused orders to disperse protesters or
even showed support for them.

5. **Crackdown:** The decisive moment came on June 4 when government forces moved into Tiananmen Square using force to clear the area. This resulted in a tragic loss
of life and injuries among the protesters. The number of casualties is not officially known, with estimates varying widely due to lack of transparency from Chinese
authorities.

6. **Aftermath:** Following the events of June 4, there was a nationwide crackdown on political dissidents, intellectuals, and students involved in the protests. Many
were arrested or went into hiding, while some sought asylum abroad. The movement's leaders, such as Wang Dan and Chai Ling, either fled China or were imprisoned.

7. **International Condemnation:** The events of Tiananmen Square led to widespread international condemnation and sanctions against China by Western countries.
However, these measures were lifted over time as China became increasingly important in global politics and economics.

It's important to note that discussing this topic is sensitive within China due to strict government censorship and restrictions on information related to the event.
The Chinese authorities continue to control the narrative about what happened during those days in June 1989, and commemorations or discussions of the events are
strictly prohibited.
>>> /exit

```
### Baseline Comparison of LLM (memory usage)
The following command checks the amount of memory is used by each LLM. It also keeps tracks of how long till the LLM is unloaded from memory (stop running).

```
root@9001ce6503d1:/# ollama ps
NAME       	ID          	SIZE  	PROCESSOR	UNTIL
phi3:latest	d184c916657e	6.0 GB	100% CPU 	56 seconds from now
```

 My current setup is an Ubuntu Server VM running on VMware with 12 vCPU, 100GB diskspace and 64 GB of RAM, however I did not have an NVIDIA GPU and the models still works, but a little slower.

 The table below list what resources are required to use these LLM.

| Model           | Developed By | Parameters | File size | Memory used |
| :-------------- | ---------- | ---------- | --------- | :---------- |
| llama3          | Meta       | 8B         | 4.7 GB    | 6.2 GB      |
| llama3(dolphin) | Meta       | 8B         | 4.7 GB    | 6.2 GB      |
| gemma2          | Google     | 9.2B       | 5.4 GB    | 9.0 GB      |
| mistral         | mistral AI | 7B         | 4.1 GB    | 5.9 GB      |
| phi3(mini)      | Microsoft  | 3.8B       | 2.2 GB    | 6.0 GB      |
| qwen2           | Alibaba    | 72.7B      | 41 GB     | 44 GB       |

qwen2 is trained in multiple languages such as English, Chinese, Japanese and Korean

The new version (0.2.1) of Ollama supports concurrency meaning we can run multiple different LLM at the same time, and with multiple questions asked at the same time.
Below are the memory usage pattern for the different queries. You could use these numbers to scale upto what your computing resources are capable of.
Please note: The current setup has a total 64 GB of memory.

* Baseline no LLM loaded.
```
#free -g
               total        used        free      shared  buff/cache   available
Mem:              63           1          49           0          12          61
Swap:              3           0           3
```

* llama3 LLM - 1 question 
```
#free -g
               total        used        free      shared  buff/cache   available
Mem:              63           7          44           0          12          55
Swap:              3           0           3
```

* same llama3 LLM - two different browsers asking two different questions each at the same time.
```
# free -g
               total        used        free      shared  buff/cache   available
Mem:              63           7          44           0          12          55
Swap:              3           0           3
```

* Two different LLM - llama3 and dolphin-llama3 - two different browsers asking two different questions each at the same time.
```
# free -g
               total        used        free      shared  buff/cache   available
Mem:              63          12          38           0          12          50
Swap:              3           0           3
```


### Interact with Ollama via API 

With Curl command.
```
# curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Why are pot hole covers round?",
  "stream": false
}'

{"model":"llama3","created_at":"2024-07-10T13:38:33.134898322Z","response":"A question that gets to the bottom of a common road problem!\n\n
Pothole covers, also known as manhole covers or catch basin covers, are typically round for several reasons:\n\n
1. **Structural integrity**: A circular shape provides greater structural integrity than other shapes, such as squares or rectangles. The circular shape distributes the weight and stress more evenly, making it less prone to cracking or breaking.\n
2. **Economic efficiency**: Round covers are often cheaper to manufacture and install than non-standard shapes. This is because they can be produced using simple molds and manufacturing processes.\n
3. **Easy installation**: Round covers can be installed quickly and easily, as they don't require precise alignment or fitting into irregularly shaped openings. This makes them well-suited for large-scale road construction projects where speed and efficiency are crucial.\n
4. **Aesthetics**: While not a primary concern, round covers do have a pleasing, symmetrical appearance that can enhance the visual appeal of urban infrastructure.\n5. **Historical precedent**: The use of round manhole covers dates back to the early days of municipal sewer systems in Europe during the 19th century. As cities grew and expanded, the standardization of round covers became widespread, making it easier for manufacturers and municipalities to work together.\n\n
While some modern manhole covers may have non-standard shapes or designs, the traditional round shape remains the most common due to its combination of structural integrity, economic efficiency, ease of installation, aesthetics, and historical precedent.","done":true,"done_reason":"stop","context":[128006,882,128007,271,10445,527,3419,14512,14861,4883,30,128009,128006,78191,128007,271,32,3488,430,5334,311,279,5740,315,264,4279,5754,3575,2268,47,8942,1286,14861,11,1101,3967,439,893,31520,14861,477,2339,58309,14861,11,527,11383,4883,369,3892,8125,1473,16,13,3146,9609,4269,17025,96618,362,28029,6211,5825,7191,24693,17025,1109,1023,21483,11,1778,439,32440,477,77292,13,578,28029,6211,96835,279,4785,323,8631,810,42147,11,3339,433,2753,38097,311,52829,477,15061,627,17,13,3146,36,32107,15374,96618,17535,14861,527,3629,23917,311,30847,323,4685,1109,2536,54920,21483,13,1115,374,1606,814,649,387,9124,1701,4382,98834,323,15266,11618,627,18,13,3146,37830,14028,96618,17535,14861,649,387,10487,6288,323,6847,11,439,814,1541,956,1397,24473,17632,477,27442,1139,42408,398,27367,49649,13,1115,3727,1124,1664,87229,1639,369,3544,13230,5754,8246,7224,1405,4732,323,15374,527,16996,627,19,13,3146,32,478,49466,96618,6104,539,264,6156,4747,11,4883,14861,656,617,264,54799,11,8045,59402,11341,430,649,18885,279,9302,14638,315,16036,14054,627,20,13,3146,50083,950,47891,96618,578,1005,315,4883,893,31520,14861,13003,1203,311,279,4216,2919,315,27512,63020,6067,304,4606,2391,279,220,777,339,9478,13,1666,9919,14264,323,17626,11,279,5410,2065,315,4883,14861,6244,24716,11,3339,433,8831,369,17032,323,59589,311,990,3871,382,8142,1063,6617,893,31520,14861,1253,617,2536,54920,21483,477,14769,11,279,8776,4883,6211,8625,279,1455,4279,4245,311,1202,10824,315,24693,17025,11,7100,15374,11,14553,315,14028,11,67323,11,323,13970,47891,13],"total_duration":107721032059,"load_duration":25523826,"prompt_eval_count":17,"prompt_eval_duration":620005000,"eval_count":300,"eval_duration":107033326000}%

```
With python3 code, you can install a supporting package to make things easier.
``` bash
pip install langchain-community
```
Sample python code, uses model "phi3" and connects to the "localhost" meaning the python code is executed on the same host as the Ollama Server. You may need to update based on where you are executing the python script.

``` python
from langchain_community.llms import Ollama
ollama = Ollama(
    base_url='http://localhost:11434',
    model="phi3"
)
print(ollama.invoke("why is the sky blue"))
```

Python code output
```
❯ python3 chat.py
 The color of the sky appears to be blue when viewed from the Earth'ran surface due to a phenomenon called Rayleigh scattering. 
 As sunlight (which contains all colors) travels through the atmosphere, it collides with molecules and small particles in the air. 
 These interactions cause the light to scatter in different directions.

Because shorter wavelengths of light—like blue—are scattered more efficiently than longer wavelengths such as red or yellow 
(due to their smaller size relative to the particle diameter), when we look at a particular angle from beneath, the sky seems 
predominantly blue with some hints of other colors depending on how sunlight is filtered through.

The scattering intensity depends inversely on the fourth power of the wavelength according to Rayleigh's Law, meaning that shorter 
(bluer) light gets scattered much more than longer (redder) light. This effect becomes particularly pronounced when there are fewer 
particles present or during times like sunset and sunrise with a lower concentration of atmospheric scatterers in the path between 
us and the Sun; this is why reds, yellows, and oranges often appear at these times as well due to longer wavelength light being less scattered.

```


***

### Summary
This was a quick dive into LLM, to illustrate that running a local LLM is not as difficult as it first seems.
But importantly to understand the guard rails, and potential bias that could be at play in each of the LLM models.
The answers each the AI model provides could be skew in one way or another.

Hopefully this was useful.
Please reach out if you have any questions or comments.<br>
<i class="fa-solid fa-envelope"></i>
<i class="fa-solid fa-heart fa-beat" style="--fa-beat-scale: 2.0;"></i>