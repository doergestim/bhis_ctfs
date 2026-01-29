<a name="setup"></a>**Setup**

For this lab you’ll need a running instance of dvmcp, a local MCP client that can connect to a MCP server and use **tools** and **resources** and mcp-inspector. You can choose [any of the compatible](https://modelcontextprotocol.io/clients#feature-support-matrix), Below you can find setup instructions for dvmcp and Claude desktop or Goose for the mcp client, you can choose the one that you prefer.

List of MCP clients along with compatibility matrix:

https://modelcontextprotocol.io/clients#feature-support-matrix

<a name="dvmcp"></a>**Dvmcp**

First you need to have [git](https://git-scm.com/) and [docker](https://www.docker.com/) installed and running on your system.

https://git-scm.com/

https://www.docker.com/

You can clone the repo by running

<a name="cb1"></a>git clone https://github.com/harishsg993010/damn-vulnerable-MCP-server\
cd damn-vulnerable-MCP-server

Then you need to build and run the docker image containing all MCP servers by running:

<a name="cb2"></a>docker build -t dvmcp .\
docker run -p 9001-9010:9001-9010 dvmcp

<a name="claude-desktop"></a>**Claude desktop**

First install [claude desktop](https://claude.com/download) (windows and MacOS only)

https://claude.com/download

Then install [nodejs](https://nodejs.org/en) and run command below

https://nodejs.org/en

<a name="cb3"></a>npm install -g mcp-remote

Note that on some systems you might be prompted for administrative access, it that case, you can use sudo npm install -g mcp-remote

Go to settings

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.001.png)

Edit config

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.002.png)

And paste this configuration into the claude\_desktop\_config.json file

<a name="cb4"></a>{\
`  `"mcpServers": {\
`    `"Challenge 1": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9001/sse"\
`      `]\
`    `},\
`    `"Challenge 2": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9002/sse"\
`      `]\
`    `},\
`    `"Challenge 3": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9003/sse"\
`      `]\
`    `},\
`    `"Challenge 4": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9004/sse"\
`      `]\
`    `},\
`    `"Challenge 5": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9005/sse"\
`      `]\
`    `},\
`    `"Challenge 6": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9006/sse"\
`      `]\
`    `},\
`    `"Challenge 7": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9007/sse"\
`      `]\
`    `},\
`    `"Challenge 8": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9008/sse"\
`      `]\
`    `},\
`    `"Challenge 9": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9009/sse"\
`      `]\
`    `},\
`    `"Challenge 10": {\
`      `"command": "npx",\
`      `"args": [\
`        `"mcp-remote",\
`        `"http://127.0.0.1:9010/sse"\
`      `]\
`    `}\
`  `}\
}

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.003.png)

After that you should be able switch the MCPs on and and start to proceed with the challenges.

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.004.png)

After that you should be able to start the chat and proceed with the challenges.

<a name="goose"></a>**Goose**

First install [goose](https://block.github.io/goose/) (windows, MacOS, linux, different model providers including local with ollama)

https://block.github.io/goose/

I’ll showcase how to get the local setup with ollama, but it likely supports your favourite LLM provider either through API keys or open router. It’ll be very similar.

In order to get the local setup you need to install [ollama](https://ollama.com/) and make sure that it’s running (sudo systemctl status ollama for linux with systemd). If you’re not sure if ollama is running you can always run this command

<https://ollama.com/>

<a name="cb5"></a>ollama serve

If this command returns Error: listen tcp 127.0.0.1:11434: bind: address already in use that means it’s allready running.

Then you need to pull a model. For this setup I’ll use qwen3:14b, you can see a list of available models at [here](https://ollama.com/search). **be aware that not all of them will support mcp**. Try to choose newer models **with tools tag**.

Also note that you need accelerated hardware that [ollama supports](https://docs.ollama.com/gpu#hardware-support) in order to get the model running locally, as well as as enough VRAM to fit model + context.

Generally you should have a few a few additional gigabytes additionally to the sze of model weights. If you’re still unsure, you can find the quantized model on hugginface and use their calculator. In order to do that you need to sign in and add your hardware in *profile* > *edit profile* > *local apps and hardware*. Then you should be able to find a quantized model (GGUF most of the time) and see a mark next to each quantization like this (example for Qwen3-14b and rtx4080 mobile): ![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.005.png)

If you don’t have access to that kind of hardware, you probably should use an API provider with goose or claude desktop instead.

You can download the model by running

<a name="cb6"></a>ollama pull qwen3:14b

Then let’s proceed with goose

First select *Other providers* > *Ollama* > *Configure*

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.006.png)

Sumit it with default value localhost

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.007.png)

Next you should see a green checkmmark next to ollama provider along with a launch button which means all is correct

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.008.png)

Next you’ll need to select the right model.

In order to do that go to *Settings* tab > *Switch models*, select Ollama and paste your model name in the box below and click *Select model*.

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.009.png)

You should see a green message box if it’s all good.

After that you can start adding MCP servers for the challenges.

Go to the *Extensions* tab and click *Add custom extension*

Then fill all other the data for all extensions, make sure the *Type* is set to SSE.

|Challenge number|Endpoint|
| :- | :- |
|Chal 1|http://127.0.0.1:9001/sse|
|Chal 2|http://127.0.0.1:9002/sse|
|Chal 3|http://127.0.0.1:9003/sse|
|Chal 4|http://127.0.0.1:9004/sse|
|Chal 5|http://127.0.0.1:9005/sse|
|Chal 6|http://127.0.0.1:9006/sse|
|Chal 7|http://127.0.0.1:9007/sse|
|Chal 8|http://127.0.0.1:9008/sse|
|Chal 9|http://127.0.0.1:9009/sse|
|Chal 10|http://127.0.0.1:9010/sse|

You can switch the extension on and off in the *Extensions* tab based on which challenge are you working on right now.

![](Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.010.png)

After that you can go to new chat by pressing *File* > *New chat* and procees with the challenges

<a name="mcp-inspector"></a>**Mcp inspector**

For some of the challenges you might also need [mcp-inspector](https://modelcontextprotocol.io/docs/tools/inspector)

https://modelcontextprotocol.io/docs/tools/inspector

You can install it by running:

<a name="cb7"></a>npm install -g @modelcontextprotocol/inspector

![ref1]

![ref1]

<a name="challanges"></a>**Challanges**

1. Can you access the API Key hidden on the MCP server in challenge 1? Answer: sk-a1b2c3d4e5f6g7h8i9j0
1. What’s the quisition target in challenge 2? Answer: SmallTech Inc
1. What’s the prod database password in challenge 3? Answer: Pr0d-DB-S3cret!
1. What’s the AWS access key in challenge 4? Answer: AKIA5EXAMPLE12345678
1. What’s the API key in challenge 5? Answer: sk\_live\_51NxEcTGj8ZkHytYqEGBwZfmQQCxVbVQzKlKjkLmn
1. What’s the production payment gateway API key in challenge 6? Answer: pg\_live\_9i8u7y6t5r4e3w2q
1. What’s the email service token in challenge 7? Answer: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFDTUUgQ29ycCIsImVtYWlsX2FjY2VzcyI6InRydWUiLCJpYXQiOjE2NTE4ODQ4MDB9.5TxfEAVbZRGbKnXaL9Lrx9NqXSNFw2ac4FDzPHvRzSQ
1. What’s the AWS secret API key in challenge 8? Answer: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
1. What’s the admin token in challenge 10? Answer: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbiIsIm5hbWUiOiJBZG1pbiBVc2VyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNjUxODg0ODAwfQ.8FhJ7Z5KFUEJFoQW2xeUL9\_NOzlKB3j8fKvxU\_5qB4Y

[ref1]: Aspose.Words.40e5ccd1-cb28-4faa-9a40-5c1ef26920a6.011.png
