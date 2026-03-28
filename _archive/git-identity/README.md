---
type: spec
status: active
version: 0.2.0
created: 2026-03-07
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Git Identity Protocol — Multi-Repo Attribution

> A convention for managing git author identity across multiple repositories, organizations, and automation accounts — ensuring human work is attributed to humans and bot work is clearly marked.

## 1. The Problem

When you operate across multiple GitHub accounts, orgs, and automated agents, git's global `user.name` / `user.email` config silently misattributes commits. You push to your personal repo and it shows up as your company bot. Your open source contribution graph is empty because everything is credited to a service account.

This compounds when AI agents start making commits on your behalf — their work should be visible but distinguishable from yours.

## 2. Identity Zones

Define every identity that commits to your repositories:

| Zone | Type | Used For |
|------|------|----------|
| **personal** | human | Personal repos, open source, your own projects |
| **business** | org | Client work, company repos, internal tools |
| **agent** | bot | Automated commits (maintenance, memory, scheduled tasks) |

### 2.1 Rules

- **Human work → human identity.** If you wrote it or directed an agent to write it, your name goes on it.
- **Bot work → bot identity.** If an agent wrote it autonomously (scheduled tasks, auto-updates), use the agent identity.
- **Agent-assisted human work → human identity.** If you directed an agent to write code in a session, that's your commit. You're the author. The agent is the tool.
- **Never let a bot identity land on personal repos by accident.** That's what this protocol prevents.

## 3. Path-Based Resolution

Git's `includeIf` maps directory paths to identity configs:

```gitconfig
# ~/.gitconfig

[user]
    name = Your Company
    email = team@company.com

# Personal repos override
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal

# Product repos — you're the author
[includeIf "gitdir:~/workspace/products/"]
    path = ~/.gitconfig-personal
```

```gitconfig
# ~/.gitconfig-personal

[user]
    name = Your Name
    email = you@example.com
```

### 3.1 Why Global Defaults to Business

The global default should be the **business** identity because:
- Client/company repos are often the most common workspace
- Business is the safe default — you actively opt-in to personal attribution
- Forgetting to configure means commits go to the org (recoverable), not the other way around

### 3.2 Per-Repo Overrides

For repos that don't fit the path convention:

```bash
cd /path/to/special-repo
git config user.name "Your Name"
git config user.email "you@example.com"
```

## 4. Agent Commits

When an agent makes **autonomous** commits — scheduled maintenance, memory crystallization, automated PRs — use a dedicated bot identity:

```bash
git -c user.name="Agent" -c user.email="agent@example.com" commit -m "chore: crystallize memory"
```

### 4.1 Co-Authored Commits

When a human directs an agent to write code, the human is the author. Optionally credit the agent:

```
feat: add protocol heat tracking

Co-authored-by: Agent <agent@example.com>
```

### 4.2 GitHub App Commits

For GitHub Apps that push commits (CI bots, dependabot-style):

```
Author: your-app[bot] <123456+your-app[bot]@users.noreply.github.com>
```

GitHub recognizes the `[bot]` suffix and shows the bot badge automatically.

## 5. Email Registration

Every email used for commits must be added to the correct GitHub account. If an email isn't registered, GitHub can't link the commit to a profile. The contribution graph stays empty.

## 6. Verification (Optional)

For public repos where authorship matters, sign commits with GPG or SSH:

```gitconfig
[user]
    signingkey = <key-id>
[commit]
    gpgsign = true
```

This proves the commit actually came from the claimed author — important when agents can impersonate anyone via git config.

## 7. Audit

Check attribution before pushing:

```bash
# Who am I in this repo?
git config user.name && git config user.email

# Who authored recent commits?
git log --format="%ae %an" -5

# Find misattributed commits (before push)
git commit --amend --author="Name <email>" --no-edit
```

## 8. Attribution

```
This project uses the Git Identity Protocol
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Git Identity Protocol v0.2 — Curtis Mercier — CC BY 4.0*
