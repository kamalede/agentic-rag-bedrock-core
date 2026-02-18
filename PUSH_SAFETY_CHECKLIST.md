# Pre-push safety checklist

Use this before pushing to GitHub.

---

## 1. Secrets and `.env`

- [ ] **`.env` is in `.gitignore`** — Already done. It contains your real API keys and must **never** be committed.
- [ ] If you ever committed `.env` before adding it to `.gitignore`, run:
  ```bash
  git rm --cached .env
  ```
  (This stops tracking `.env` but keeps the file on your machine.)
- [ ] **Rotate any keys** that were ever committed or pasted in chat/terminals (Groq, Hugging Face). Create new keys and update your local `.env` only.

---

## 2. What gets pushed

- [ ] Run and review:
  ```bash
  git status
  git diff --staged
  ```
- [ ] Confirm **no** `.env`, `.venv/`, or real API keys appear in the file list or diffs.

---

## 3. Your code (no hardcoded secrets)

- [ ] **00_langgraph_agent.py**, **01_agentcore_runtime.py**, **02_agentcore_memory.py** — All use `os.getenv("GROQ_API_KEY")` (or similar). No hardcoded API keys. **OK.**

---

## 4. Config and AWS details

- [ ] **`.bedrock_agentcore.yaml`** — Contains your AWS account ID and ARNs. If the repo is **public** and you prefer not to expose that, either:
  - Add `.bedrock_agentcore.yaml` to `.gitignore` and commit a `.bedrock_agentcore.example.yaml` with placeholders, or  
  - Redact account ID / ARNs before pushing.  
  If the repo is **private**, many people leave this file as-is.

---

## 5. Sample env only

- [ ] **`.sample_env`** — Contains only placeholders (`<YOUR GROQ API KEY>`). Safe to commit. **OK.**

---

## 6. Quick commands (first-time push)

```bash
cd C:\Users\kamal\Agentcore
git init
git add .
git status
# Ensure .env and .venv do NOT appear
git commit -m "Initial commit: AgenticAI"
git remote add origin <your-github-repo-url>
git push -u origin main
```

---

**Summary:** Your `.gitignore` already excludes `.env`, `.venv/`, and other sensitive/noise files. Just confirm `.env` is never staged and rotate any keys that were ever exposed.
