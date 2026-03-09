---
type: spec
status: active
version: 0.1.0
created: 2026-03-07
updated: 2026-03-07
author: Curtis Mercier
license: CC BY 4.0
---

# Git Identity Protocol — Multi-Repo Attribution

> A convention for managing git author identity across multiple repositories, organizations, and automation accounts — ensuring human work is attributed to humans and bot work is clearly marked.

## 1. The Problem

When you operate across multiple GitHub accounts, orgs, and automated agents, git's global `user.name` / `user.email` config silently misattributes commits. You push to your personal repo and it shows up as your company bot. Your open source contribution graph is empty because everything is credited to a service account.

This compounds when AI agents start making commits on your behalf — their work should be visible but distinguishable from yours.

## 2. Identity Table

Define every identity that commits to your repositories:

| Identity | Name | Email | Type | Used For |
|----------|------|-------|------|----------|
| **personal** | Curtis Mercier | github@gravicity.ai | human | Personal repos, protocols, open source, meetsoma |
| **business** | Gravicity | accounts@gravicity.ca | org | Client work, infra, internal tools |
| **agent** | Soma | soma-agent@gravicity.ai | bot | Automated commits by Soma (maintenance, memory, scheduled) |

### 2.1 Rules

- **Human work → human identity.** If you wrote it, your name goes on it.
- **Bot work → bot identity.** If an agent wrote it autonomously (scheduled tasks, auto-updates), use the agent identity.
- **Agent-assisted human work → human identity.** If you directed an agent to write code in a session, that's your commit. You're the author. The agent is the tool.
- **Never let a bot identity land on personal repos by accident.** That's what this protocol prevents.

## 3. Path-Based Resolution

Git's `includeIf` maps directory paths to identity configs:

```gitconfig
# ~/.gitconfig

[user]
    name = Gravicity
    email = accounts@gravicity.ca

# Personal repos override
[includeIf "gitdir:~/Gravicity/personal/"]
    path = ~/.gitconfig-personal

# Product repos (meetsoma) — you're the author
[includeIf "gitdir:~/Gravicity/products/soma/"]
    path = ~/.gitconfig-personal

# Any repo cloned under ~/personal/ or ~/oss/
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

```gitconfig
# ~/.gitconfig-personal

[user]
    name = Curtis Mercier
    email = github@gravicity.ai
```

### 3.1 Why Global Defaults to Business

The global default is the **business** identity because:
- Client repos (`clients/`) are the most common workspace
- Infra repos should be attributed to the org
- Business is the safe default — you actively opt-in to personal attribution
- Forgetting to configure means commits go to the org (recoverable), not the other way around

### 3.2 Per-Repo Overrides

For repos that don't fit the path convention:

```bash
cd /path/to/special-repo
git config user.name "Curtis Mercier"
git config user.email "github@gravicity.ai"
```

This lives in `.git/config` and overrides everything.

## 4. Agent Commits

When Soma (or any agent) makes **autonomous** commits — scheduled maintenance, memory crystallization, automated PRs — use a dedicated bot identity:

```bash
git -c user.name="Soma" -c user.email="soma-agent@gravicity.ai" commit -m "chore: crystallize memory muscles"
```

### 4.1 Co-Authored Commits

When a human directs an agent to write code, the human is the author. Optionally credit the agent as co-author:

```
feat: add protocol heat tracking

Co-authored-by: Soma <soma-agent@gravicity.ai>
```

### 4.2 GitHub App Commits

For GitHub Apps that push commits (CI bots, dependabot-style):

```
Author: gravicity-app[bot] <123456+gravicity-app[bot]@users.noreply.github.com>
```

GitHub recognizes the `[bot]` suffix and shows the bot badge automatically.

## 5. Email Registration

Every email used for commits must be added to the correct GitHub account:

| Email | Add To Account | Purpose |
|-------|----------------|---------|
| `github@gravicity.ai` | `curtismercier` | Personal commits show on your profile |
| `accounts@gravicity.ca` | `gravicity-archive` or org | Business commits |
| `soma-agent@gravicity.ai` | (future bot account) | Agent commits |

**If an email isn't registered**, GitHub can't link the commit to a profile. The contribution graph stays empty.

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

# Find misattributed commits
git log --format="%ae %an %s" | grep -v "github@gravicity.ai" | grep -v "accounts@gravicity.ca"
```

### 7.1 Fix Misattributed Commits (Before Push)

```bash
# Amend the last commit
git commit --amend --author="Curtis Mercier <github@gravicity.ai>" --no-edit

# Rewrite last N commits
git rebase -i HEAD~N
# mark commits as "edit", then:
git commit --amend --author="Curtis Mercier <github@gravicity.ai>" --no-edit
git rebase --continue
```

### 7.2 Fix Misattributed Commits (After Push)

```bash
git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "wrong@email.com" ]; then
    export GIT_AUTHOR_NAME="Curtis Mercier"
    export GIT_AUTHOR_EMAIL="github@gravicity.ai"
    export GIT_COMMITTER_NAME="Curtis Mercier"
    export GIT_COMMITTER_EMAIL="github@gravicity.ai"
fi' HEAD

git push --force
```

⚠️ Force-push rewrites history. Only do this on repos where you're the sole contributor or with team coordination.

## 8. Attribution

```
This project uses the Git Identity Protocol
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Git Identity Protocol v0.1 — Curtis Mercier — CC BY 4.0*
