# 🔐 Universal SSH Setup

**Windows 11 · macOS · Linux · Docker · Multiple Accounts**

This guide shows the correct, modern way to set up SSH so it works across Windows, macOS, Linux, Docker, GitHub, and GitLab — including multiple accounts.

---

## 🧠 What You'll Achieve

| Account         | Host Alias        | Key File                     | Service    |
| --------------- | ----------------- | ---------------------------- | ---------- |
| Personal GitHub | `github-personal` | `id_ed25519_github_personal` | github.com |
| Work GitHub     | `github-work`     | `id_ed25519_github_work`     | github.com |
| GitLab          | `gitlab-personal` | `id_ed25519_gitlab_personal` | gitlab.com |

---

## 1️⃣ Check OpenSSH

### Windows (PowerShell)

```powershell
ssh -V
```

Install via Optional Features if missing.

### macOS / Linux

```bash
ssh -V
```

---

## 2️⃣ Generate Separate SSH Keys (per account)

```bash
ssh-keygen -t ed25519 -C "you@gmail.com" -f ~/.ssh/id_ed25519_github_personal
ssh-keygen -t ed25519 -C "you@company.com" -f ~/.ssh/id_ed25519_github_work
ssh-keygen -t ed25519 -C "you@gmail.com" -f ~/.ssh/id_ed25519_gitlab_personal
```

Windows path example:

```bash
ssh-keygen -t ed25519 -C "you@gmail.com" -f C:/Users/username/.ssh/id_ed25519_github_personal
```

---

## 3️⃣ Start SSH Agent & Add Keys

### Windows

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

ssh-add $env:USERPROFILE\.ssh\id_ed25519_github_personal
ssh-add $env:USERPROFILE\.ssh\id_ed25519_github_work
ssh-add $env:USERPROFILE\.ssh\id_ed25519_gitlab_personal
```

### macOS / Linux

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github_personal
ssh-add ~/.ssh/id_ed25519_github_work
ssh-add ~/.ssh/id_ed25519_gitlab_personal
```

---

## 4️⃣ Add Public Keys to GitHub / GitLab

```bash
cat ~/.ssh/id_ed25519_*.pub
```

Add to each service's SSH keys settings.

---

## 5️⃣ Create or Edit the SSH Config File (Critical)

### Location

| OS            | Path                            |
| ------------- | ------------------------------- |
| Windows       | `C:\Users\YourName\.ssh\config` |
| macOS / Linux | `~/.ssh/config`                 |

### Create / Edit

Windows:

```powershell
notepad $env:USERPROFILE\.ssh\config
```

macOS / Linux:

```bash
nano ~/.ssh/config
```

### Example Config

```ssh
# GitHub Personal
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_personal
    IdentitiesOnly yes

# GitHub Work
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_work
    IdentitiesOnly yes

# GitLab
Host gitlab-personal
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab_personal
    IdentitiesOnly yes
```

---

## 6️⃣ Clone Using Aliases

```bash
git clone git@github-personal:username/repo.git
git clone git@github-work:company/repo.git
git clone git@gitlab-personal:username/repo.git
```

---

## 7️⃣ Fix Existing Repositories

```bash
git remote set-url origin git@github-work:company/repo.git
```

---

## 8️⃣ Set Git Identity Per Repo

```bash
git config user.name "Your Name"
git config user.email "you@gmail.com"
```

---

## 9️⃣ Test Connections

```bash
ssh -T git@github-personal
ssh -T git@github-work
ssh -T git@gitlab-personal
```

---

## 🔟 Docker & SSH

```bash
docker run -it -v ~/.ssh:/root/.ssh:ro ubuntu:latest bash
ssh -T git@github-personal
```

---

# 🧯 Troubleshooting SSH Connection Issues (Windows)

### ❌ Error: `Permission denied (publickey)` but `ssh -T` works

**Cause:** Git is using a different SSH or wrong key.

**Fix:** Ensure `IdentitiesOnly yes` and correct `IdentityFile` in `config`. Then clone using the alias.

---

### ❌ Error: `Bad owner or permissions on .ssh/config`

```powershell
cd $env:USERPROFILE\.ssh
icacls config /inheritance:r
icacls config /grant:r "$($env:USERNAME):(F)"
icacls config /remove "Users"
icacls config /remove "Everyone"
```

---

### ❌ Debug which key Git is using

```powershell
git -c core.sshCommand="ssh -v" clone git@github-personal:username/repo.git
```

Look for `Offering public key` lines.

---

### ❌ SSH tries default keys (`id_rsa`, `id_ed25519`)

Add `IdentitiesOnly yes` and use host aliases.

---

### ❌ Multiple GitHub accounts conflict

Never use `Host github.com` twice. Use aliases like `github-work`, `github-personal`.

---
