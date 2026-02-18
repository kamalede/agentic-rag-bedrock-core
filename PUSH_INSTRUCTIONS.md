# Push to GitHub: agentic-rag-bedrock-core

## Step 1: Create the repository on GitHub

1. Open this link (creates a new repo with the right name):
   **https://github.com/new?name=agentic-rag-bedrock-core**
2. Leave **"Add a README"** unchecked (you already have one).
3. Click **Create repository**.

## Step 2: Add remote and push (PowerShell)

Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username, then run:

```powershell
cd C:\Users\kamal\Agentcore

git remote add origin https://github.com/YOUR_GITHUB_USERNAME/agentic-rag-bedrock-core.git
git branch -M main
git push -u origin main
```

If you use **SSH** instead of HTTPS:

```powershell
git remote add origin git@github.com:YOUR_GITHUB_USERNAME/agentic-rag-bedrock-core.git
git push -u origin main
```

## Step 3: If Git asks for login

- **HTTPS:** Use your GitHub username and a **Personal Access Token** (not your password).
  - Create one: GitHub → Settings → Developer settings → Personal access tokens.
- **SSH:** Ensure your SSH key is added to your GitHub account.

---

Your initial commit is already done. Only Step 1 (create repo) and Step 2 (add remote + push) are left.
