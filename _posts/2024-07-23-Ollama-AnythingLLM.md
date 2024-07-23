---
title: "RAG + Embedding with AnythingLLM and Ollama"
layout: single
tags:
  - docker
  - ollama
  - llm
  - anythingllm
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: Using Ollama with AnythingLLM enhances the capabilities of your local Large Language Models (LLMs) by providing a suite of functionalities that are particularly beneficial for private and sophisticated interactions with documents.
categories: posts
sitemap: true
pkeywords: docker, ollama, llm, python, anythingLLM,
---
This blog post is to demonstrate how easy and accessible RAG capabilities are when we leverage the strengths of AnythingLLM and Ollama to enable Retrieval-Augmented Generation (RAG) capabilities for various document types.

AnythingLLM - is an all-in-one AI application that simplifies the interaction with Large Language Models (LLMs) for business intelligence purposes. It allows users to chat with any document, such as PDFs or Word files, using various LLMs, including enterprise models like GPT-4 or open-source models like Llama and Mistral. 

Ollama - we introduce this in the last blog. We will use its ability to service multiple Open Source LLMs via its API interface.

Here is a quick outline:
* Details about docker containers for Ollama (Platform/Server) and AnythingLLM (front end/chat/uploading documents).
* Explore:
	* Embedding a news article about recent US political events and see how we it will do.
    * Embedding LLM 
	* Vector database (Lance DB) - you can spin chroma if you like, but Lance DB comes bundled with ANything LLM.

 A lot of the above is built into AnythingLLM.

***
### Components used
* [Ollama Server](https://ollama.com/) - a platform that make easier to run LLM locally on your compute.
* [Open WebUI](https://github.com/open-webui/open-webui) - a self-hosted front end that interacts with APIs that presented by Ollama or OpenAI compatible platforms. I am using to download new LLMs much easier to manage than connecting to the ollama docker container and issuing 'ollama pull'.
* [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) - an all-in-one AI application that simplifies the interaction with Large Language Models (LLMs).
* Linux Server or equivalent device - spin up three docker containers with the Docker-compose YAML file specified below.

***
### Code break down
> Analysis of Docker-Compose.yml file

{% highlight yaml linenos %}
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

  anything-LLM:
    image: mintplexlabs/anythingllm:latest
    container_name: anything-llm
    cap_add:
      - SYS_ADMIN
    restart: unless-stopped
    environment:
      - SERVER_PORT=3001
      - UID='1000'
      - GID='1000'
      - STORAGE_DIR=/app/server/storage
      - LLM_PROVIDER=ollama
      - OLLAMA_BASE_PATH=http://ollama-server:11434
      - OLLAMA_MODEL_PREF='phi3'
      - OLLAMA_MODEL_TOKEN_LIMIT=4096
      - EMBEDDING_ENGINE=ollama
      - EMBEDDING_BASE_PATH=http://ollama-server:11434
      - EMBEDDING_MODEL_PREF=nomic-embed-text:latest
      - EMBEDDING_MODEL_MAX_CHUNK_LENGTH=8192
      - VECTOR_DB=lancedb
      - WHISPER_PROVIDER=local
      - TTS_PROVIDER=native
      - PASSWORDMINCHAR=8
    volumes:
      - ./anythingllm_data/storage:/app/server/storage
      - ./anythingllm_data/collector/hotdir/:/app/collector/hotdir
      - ./anythingllm_data/collector/outputs/:/app/collector/outputs
    ports:
      - "3001:3001"
    extra_hosts:
      - host.docker.internal:host-gateway
{% endhighlight %}

*Line 6* - Ollama Server exposes port 11434 for its API.

*Line 8* - maps a folder on the host ollama_data to the directory inside the container /root/.ollama - this is where all LLM are downloaded to.

*Line 16* - environment variable that tells Web UI which port to connect to on the Ollama Server. Since both docker containers are sitting on the same host we can refer to the ollama container name 'ollama-server' in the URL.

*Line 18* - maps a folder on the host webui to the directory inside the container /app/backend/data - storing configs.

*Line 20* - Connect to the Web UI on port 3010.

*Line 21-22* - Avoids the need for this container to use 'host' network mode.

*Line 30* - Environmental variable that are used by AnythingLLM - more can be found at [ENV variables](https://github.com/Mintplex-Labs/anything-llm/blob/master/docker/.env.example) Note the Base_Path to ollama refers to the ollama container listed above in the docker compose file.

*Line 47* - AnythingLLM uses a lot of volume mapping. They may make changes to this later, the last two collector was a recent addition, so it will depend on the version of the docker image that gets pulled. Since it is set to 'latest'

My directory structure in the folder where docker compose exist. Create these folders before starting the 'docker compose' commands.

```
❯ tree -L 2 -d
.
├── anythingllm_data
│   ├── collector
│   └── storage
├── ollama_data
└── webui
```

Issue 'docker compose up -d' from the folder where your docker compose file are, to install and start the containers.
Once the containers are up, you can browse to the AnythingLLM on port 3001 - example http://x.x.x.x:3001

### Check Anything LLM integration with Ollama
Open your browser and check you can get to Anything LLM, then on the bottom left, navigate to the "Open settings" button.
Check the AI Provider section for LLM that Ollama is selected and that the "Ollama Model" drop down has a list of LLM pull down already on Ollama.

[![](/assets/images/2024-07-23-Check.jpg)](/assets/images/2024-07-23-Check.jpg)

Then navigate to Embedder and check that you have 'nomic-embed-text' selected. If not use Web-Ui to download it or Ollama to pull it down.
We will use nomic-embed-text model to embed our document.

[![](/assets/images/2024-07-23-Embedder.jpg)](/assets/images/2024-07-23-Embedder.jpg)

Next checked that we are using 'LanceDB' as a Vector database.

[![](/assets/images/2024-07-23-Vector.png)](/assets/images/2024-07-23-Vector.png)

Then head back out to the main screen and create a 'New workspace' - I called mine "Trump"
Then click on the upload button to embed a document.

[![](/assets/images/2024-07-23-upload_doc.png)](/assets/images/2024-07-23-upload_doc.png)

I then uploaded a recent news article about Trump so that we can query it.

[![](/assets/images/2024-07-23-document_select.png)](/assets/images/2024-07-23-document_select.png)


### Testing the embedding.
Here are the workspace setting used.
* Gemma2 LLM 
* Chat Setting - set to Query - Query will provide answers only if document context is found.
* LLM Temperature set to 0.7

News article used was from [news.com.au](https://www.news.com.au/finance/work/leaders/us-election-live-updates-joe-biden-in-the-race-trump-rnc-speech/news-story/b340343f2947ee2242aa2127cbdc61f1)

[![](/assets/images/2024-07-23-Q1.png)](/assets/images/2024-07-23-Q1.png)

[![](/assets/images/2024-07-23-Q2.png)](/assets/images/2024-07-23-Q2.png)

[![](/assets/images/2024-07-23-Q3.png)](/assets/images/2024-07-23-Q3.png)

The last question it got in-correct - Elon did "pledged $US45 million ($52 million) monthly to a Trump super PAC"

***

### Summary
That was a quick test of AnythingLLM with Ollama. You can see it was close but could not score 3 out of 3.
It also had some issue processing PDF file, when I print page from the web browser, perhaps something with the formatting. It would accept the file but let's just say the knowledge gaps was much bigger.

Open Source Community had made AI so accessible now. Can't wait to see what the future brings.

Please reach out if you have any questions or comments.<br>
<i class="fa-solid fa-envelope"></i>
<i class="fa-solid fa-heart fa-beat" style="--fa-beat-scale: 2.0;"></i>