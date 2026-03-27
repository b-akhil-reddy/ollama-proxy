![Final output post all configurations](./assets/BlogThumbnail.png)

# 🚀 Free Hack: Turn Colab/Kaggle Notebook into Your Personal AI Backend (Ollama + Colab/Kaggle + Tunnel)

## ✨ Local AI Without Local Hardware

Running large language models locally sounds great until you hit reality:

-   Your GPU runs out of memory

-   Your laptop turns into a heater

-   Or you simply don't have the hardware

But what if you could **offload all of that to the cloud for free and still use it inside your IDE like a local model?**

That's exactly what we're going to do.

In this guide, you'll learn how to:

-   Run an **Ollama server on Google Colab or Kaggle**

-   Expose it via a public URL using a **tunnel**

-   Connect it directly to **VS Code Copilot Chat**

-   Use it for your **personal development workflows**

No paid APIs. No local GPU. Just pure engineering.

*Final output post all configurations*

[![Watch the video](./assets/VideoThumbnail.png)](https://raw.githubusercontent.com/b-akhil-reddy/ollama-proxy/main/assets/VSCodeOllama.mp4)

---

## 🔌 Beyond VS Code: You can use this Ollama Server with Other Tools

One of the biggest advantages of this setup is that you're not just creating a backend for VS Code you're spinning up a **general-purpose LLM API endpoint.**

Once your Ollama server is live (via the tunnel), **any tool that supports Ollama endpoint can use it.**

---

### 🧰 Tools You Can Plug Into

Here are a few powerful tools you can integrate right away:

### 🤖 OpenClaw / ⚙️ AgentZero

-   Autonomous agent workflows

-   Task execution and chaining

-   Can use your Ollama endpoint as the LLM backend

### 🧪 Other Compatible Tools

You can also plug into:

-   OpenAI-compatible SDKs (by redirecting base URL)

-   Custom Python scripts using requests

-   CLI tools that support HTTP LLM endpoints

This turns your setup from:

❌ "Just a VS Code integration" ➡️ into ✅ "A reusable AI backend for anything"

With this said we can start building

---

## 🧠 The Architecture (Quick Overview)

Here's what we're building:

```
 Colab/Kaggle Notebook
        ↓
   Ollama Server
        ↓
     LLM Model
        ↓
Public Tunnel (Pinggy)
        ↓
VS Code (Copilot Chat)
        or
Any other alternative
```

---

## 🧩 Part 1: Prerequisite configure VS Code (Client Side)

We'll start with the client setup so you're ready to plug in the backend later.

---

### 🔧 Install Required Tools

Install:

-   Visual Studio Code

-   GitHub Copilot Extension on VSCode

---

### ⚙️ Add a Custom Model Provider

![Chat pane showing Manage Models in Other Models Option](./assets/ManageModels.png)
*Chat pane showing Manage Models in Other Models Option*

1.  Open **Copilot Chat panel**

2.  Click the **model selector**

3.  Choose **Other Models**

4.  Click **Manage Models**

5.  Click **Add Model**

6.  Select **Ollama**

![Add model pane showing Ollama Option](./assets/CreateOllamaServer.png)
*Add model pane showing Ollama Option*

You'll be prompted for a name and then for URL leave this for now. You can come back to this after tunnel creation. You need not use only this extension any extension that supports Olamma can be used. Continue is also another extension where you can do the same.

![VSCode asking for Ollama Endpoint URL](./assets/OllamaServerConfig.png)
*VSCode asking for Ollama Endpoint URL*

---

## ☁️ Part 2: Spin Up Ollama on Colab/Kaggle (Server Side)

Now we start the backend.

You can use:

-   Google Colab

-   Kaggle Notebooks

---

### 📦 Step 1: Install Dependencies

```
!apt-get update && apt-get install -y zstd
```

Note: Most other dependencies might already be installed in Colab/Kaggle.

---

### ⚡ Step 2: Install Ollama

From:

-   Ollama

```
!curl -fsSL https://ollama.com/install.sh | sh
```

---

### ▶️ Step 3: Start the Ollama Server

In Kaggle Notebooks, shell commands executed using the `!` operator run in the foreground and block the cell until completion. This means you can't reliably use them to start long-running background processes (for example, using `&`).

To work around this limitation, you can launch the process programmatically using Python (such as with the `subprocess` module), which allows the process to run alongside your notebook without blocking execution(**when some cell is interrupted this cell must be ran again**). Here's a simple example:

```
import subprocess
subprocess.Popen([
        "/usr/local/bin/ollama", "serve"
    ],
    env={
        **os.environ,
        "OLLAMA_KEEP_ALIVE": "-1",
        "OLLAMA_HOST": "0.0.0.0:11434",
        "OLLAMA_ORIGINS": "*"
    }
)
```
![Code block starting the ollama server](./assets/OllamaStart.png)
*Code block starting the ollama server*

Verify it using the following command(you might need to install net-tools for this):

```
!netstat -tnlp
```

You should see the application running on 11434 port as following:
![Ollama server started](./assets/OllamaStarted.png)
*Ollama server started*

It is completely optional to check the status, but this helps in debugging

---

### 🤖 Step 4: Pull a Model

```
!ollama pull ministral-3:14b
!ollama pull qwen3.5:27b
```

`Note: You can pull other models that you want in this step instead of these. Make sure that the model that you are installing is tools compatible.`

![Pulling ministral-3:14b](./assets/Mistral3:14b.png)
*Pulling ministral-3:14b*

![Pulling qwen3.5:27b](./assets/Qwen3.5:27b.png)
*Pulling qwen3.5:27b*

---

## 🌐 Part 3: Expose Your Server to the Internet

Colab/Kaggle runs in isolation so we need a tunnel.

We'll use Pinggy, it is simple to configure as it works on SSH(you need to have openssh-client installed). But there are other alternatives that you can choose from: Cloudflare Tunnel(Didn't work for me on Kaggle; I kept receiving SIGINT, but it worked on Colab), Ngrok, etc.

---

### 🔗 Start the Tunnel

```
import subprocess
process = subprocess.Popen(
    [
        "ssh",
        "-v",   # or -vv / -vvv for more verbosity
        "-T",
        "-p", "443",
        "-R0:0.0.0.0:11434",
        "a.pinggy.io",
        "-o", "StrictHostKeyChecking=no",
        "-o", "UserKnownHostsFile=/dev/null"
    ],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True
)
```
![Start pinggy tunnel](./assets/FreePinggyTunnel.png)
*Start pinggy tunnel*

---

### 📡 Extract the Public URL

```
for line in process.stdout:
    print(line)
```
![Reading the logs of pinggy(I kept ssh in verbose so we have a lot of extra lines)](./assets/PinggyDebugLogs.png)
*Reading the logs of pinggy(I kept ssh in verbose so we have a lot of extra lines)*

Look for something like:

```
https://<hostname>.a.free.pinggy.link
```
![Highlighted section shows the tunnel URLs](./assets/PinggyDebugURL.png)
*Highlighted section shows the tunnel URLs*

---

### ✅ Test It

Open the URL in your browser.

You should see:

```
Ollama is running
```
![Tunnel endpoint showing 'Ollama is running'](./assets/OllamaRunning.png)
*Tunnel endpoint showing 'Ollama is running'*

You now have:

-   A portable LLM server

-   Full control over models

-   Zero API costs

-   Integration with agents, automation tools, and custom apps

---

### ⚠️ A Small Note

Since this runs on Colab/Kaggle:

-   URLs are temporary

-   No authentication by default

-   Not suitable for production

---

## 🔌 Part 4: Connect Everything Back to VS Code

Now return to VS Code:

-   Paste your Pinggy URL into the Ollama endpoint

-   Save settings

![VSCode asking for Ollama Endpoint URL](./assets/OllamaServerConfig.png)
*VSCode asking for Ollama Endpoint URL*

Within a few seconds:

-   Your models will appear

-   You can start prompting directly from Copilot Chat 🎉

![Models pane showing Configured Ollama Endpoint](./assets/OllamaConfig.png)
*Models pane showing Configured Ollama Endpoint*

---

## 💡 Pro Tips

-   Use quantized models

-   Restart clean if things hang

-   Monitor logs continuously(Ollama and ssh debug logs tell a lot about how the Ollama and tunneling work)

-   Change Ollama server default variables
**OLLAMA_CONTEXT_LENGTH, OLLAMA_BATCH_SIZE, OLLAMA_GPU_LAYERS, OLLAMA_NUM_THREADS, OLLAMA_KEEP_ALIVE**

---

## 🎯 Final Thoughts

This setup is a powerful hack:

-   ✅ No Local GPU required offload it to Colab's or Kaggle's GPU

-   ✅ No API costs

-   ✅ Fully IDE-integrated

If you take this further, you can:

-   Add authentication layers

-   Deploy on a VMs in any of the Clouds and by running them as python scripts

To make things easier, I've created a ready-to-use notebook with everything pre-configured:
-   [ollama-proxy.ipynb](./ollama-proxy.ipynb)

This Works on Google Colab / Kaggle Notebooks Includes server + tunnel + logging

This lets you get started immediately.

It's not production-ready but for **personal development and experimentation**, it's incredibly effective.