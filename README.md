# 🤖 AgenticAI

**AgenticAI** is an agentic AI system I built from scratch. It uses LangGraph, RAG over a custom FAQ knowledge base, and Amazon Bedrock AgentCore to run and deploy agents—from a simple local agent to a production agent with persistent memory on AWS.

---

## What’s in This Project

| File | Description |
|------|-------------|
| `00_langgraph_agent.py` | Basic LangGraph agent with FAQ search (local) |
| `01_agentcore_runtime.py` | AgentCore runtime agent with tool-based search and query reformulation |
| `02_agentcore_memory.py` | Agent with memory for conversation history and user preferences |
| `lauki_qna.csv` | FAQ knowledge base used by all agents |

---

## Prerequisites

- **Python** 3.13+
- **AWS account** with Bedrock access and **AWS CLI** configured
- **GROQ API key** ([console.groq.com](https://console.groq.com))
- **uv** (recommended) or pip

```bash
python --version   # 3.13 or newer
pip install uv     # or: https://docs.astral.sh/uv/getting-started/installation/
aws configure      # AWS credentials and region (e.g. us-east-1)
```

---

## Installation

**1. Clone and enter the project**

```bash
git clone <your-repo-url>
cd Agentcore
```

**2. Install dependencies**

```bash
uv sync
```

Or with pip:

```bash
pip install -e .
```

**3. Environment variables**

Copy the sample env file and add your keys (do not commit `.env`):

```bash
copy .sample_env .env   # Windows
# cp .sample_env .env  # macOS/Linux
```

Edit `.env` and set your API keys:

```env
GROQ_API_KEY=your_groq_api_key_here
HF_API_KEY=your_huggingface_api_key_here   # optional
```

---

## Running AgenticAI

### 1. Basic agent (local only)

Runs the FAQ agent locally—no AWS.

```bash
# Activate virtual environment (Windows PowerShell)
.\.venv\Scripts\Activate.ps1

# Run the agent
python 00_langgraph_agent.py
```

**Example output:**

```
Explain roaming activation.
Roaming activation allows you to use your SIM in other countries...
```

---

### 2. Deploy to Amazon Bedrock (runtime agent)

**Step 1: Configure the agent**

```bash
.\.venv\Scripts\Activate.ps1
agentcore configure -e ./01_agentcore_runtime.py
```

This creates or updates `.bedrock_agentcore.yaml` and sets the entrypoint.

**Step 2: Deploy (remote ARM64 build)**

Use the default deploy so AWS CodeBuild builds for `linux/arm64`. Do **not** use `--local` on Windows/amd64.

```bash
agentcore deploy
```

**Example deploy output:**

```
🚀 Launching Bedrock AgentCore (codebuild mode - RECOMMENDED)...
   • Build ARM64 containers in the cloud with CodeBuild
   • No local Docker required (DEFAULT behavior)

Starting CodeBuild ARM64 deployment for agent 'agentcore_lang_runtime' to account XXXXX (us-east-1)
...
✅ COMPLETED completed in 1.1s
🎉 CodeBuild completed successfully in 1m 30s
Agent created/updated: arn:aws:bedrock-agentcore:us-east-1:XXXXX:runtime/agentcore_lang_runtime-XXXXX
Deployment completed successfully
```

**Step 3: Invoke the deployed agent**

```bash
agentcore invoke '{"prompt": "Explain roaming activation"}'
```

**Example invoke output:**

```
╭──────────────────────────────────────── agentcore_lang_runtime ────────────────────────────────────────╮
│ Session: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx                                                           │
│ Request ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx                                                        │
│ ARN: arn:aws:bedrock-agentcore:us-east-1:XXXXX:runtime/agentcore_lang_runtime-XXXXX                      │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯

Response:
{"result": "Roaming activation allows you to use your SIM in other countries. To activate roaming, purchase a roaming pack in the app or enable roaming in your phone's network settings. The credentials will be pushed to the local network within minutes."}
```

---

### 3. Agent with memory (production-style)

**Step 1: Configure the memory agent**

```bash
agentcore configure -e ./02_agentcore_memory.py
```

Configure memory (STM and LTM) and link the memory resource when prompted.

**Step 2: Set default agent (optional)**

```bash
agentcore configure set-default agent_with_memory
```

**Step 3: Deploy**

```bash
agentcore deploy
```

**Step 4: Invoke**

```bash
agentcore invoke '{"prompt": "activate it for Dubai"}'
```

**Example response:**

```json
{
  "result": "**Activating your SIM for use in Dubai**\n\n1. **Insert the SIM** Put the new SIM card into your phone.\n2. **Complete KYC** During first-time activation, the app will prompt you to upload a government-issued ID and verify your address.\n3. **Verify phone number** The system will send a short code or SMS to confirm you own the device.\n4. **Restart the device** Once KYC and verification finish, reboot your phone so the network registration completes.\n\n**If you're traveling to Dubai and want roaming:**\n- Purchase a Dubai-specific roaming pack in the app or store.\n- Enable roaming in your phone's network settings.\n- The roaming credentials will be pushed to the local network within minutes.\n\nThat should get you fully activated and ready to use your service while in Dubai.",
  "actor_id": "default-user",
  "thread_id": "5ece7047-3c72-4690-945f-a0c3a83461ff"
}
```

---

## Testing in the AWS Console (Agent Sandbox)

You can test the same agent from the AWS Console.

1. Open **Amazon Bedrock** → **AgentCore** → **Test** → **Agent sandbox**.
2. Select your runtime agent (e.g. `agent_with_memory`).
3. In **Input**, enter:

   ```json
   {"prompt": "activate it for Dubai"}
   ```

4. Click **Run**. The **Output** shows the JSON response (`result`, `actor_id`, `thread_id`).

![Agent Sandbox example](assets/agent-sandbox-example.png)

> Save a screenshot of your Agent Sandbox run as `assets/agent-sandbox-example.png` to display it here.

---

## Command reference

| Goal | Command |
|------|--------|
| Configure entrypoint | `agentcore configure -e ./01_agentcore_runtime.py` |
| Set default agent | `agentcore configure set-default <agent_name>` |
| List agents | `agentcore configure list` |
| Deploy (remote build) | `agentcore deploy` |
| Invoke agent | `agentcore invoke '{"prompt": "Your question here"}'` |
| Check status | `agentcore status` |
| Local dev server | `agentcore dev` |

**Notes:**

- Use **double quotes** inside the JSON for `agentcore invoke` (e.g. `'{"prompt": "..."}'`).
- Use `agentcore deploy` (no `--local`) so the image is built for **linux/arm64** in CodeBuild.
- For `agentcore launch` with env vars: `agentcore launch --env GROQ_API_KEY=your_key`. Prefer storing secrets in `.env`.

---

## Troubleshooting

| Issue | What to do |
|-------|------------|
| **Python version** | Use 3.13+: `python --version` |
| **Missing GROQ_API_KEY** | Add `GROQ_API_KEY=...` to `.env` in project root |
| **AWS credentials** | Run `aws configure` and set region (e.g. `us-east-1`) |
| **FAISS install fails** | Try: `pip install --upgrade faiss-cpu` |
| **Platform mismatch (linux/amd64 vs arm64)** | Use `agentcore deploy` without `--local` so CodeBuild builds for arm64 |
| **No such option: -e** | Use `-e` only with `configure`: `agentcore configure -e ./file.py` |
| **Agent not deployed** | Run `agentcore deploy` and wait for “Deployment completed successfully” |

---

## Resources

- [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)
- [AgentCore documentation](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/agentcore-get-started-toolkit.html)
- [Bedrock AgentCore samples](https://github.com/awslabs/amazon-bedrock-agentcore-samples)

---

