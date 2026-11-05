---
title: GitHub Push 403 Wrong Account Incident (Diagnosis & Prevention Playbook)
tags:
  - project
  - git
  - github
  - troubleshooting
  - windows
created: 2026-11-04
status: resolved
project: "[[PMS Inventory System - Project Architecture & Requirements Proposal|PMS Inventory System]]"
---

# 🔐 GitHub Push 403 Wrong Account Incident (Diagnosis & Prevention Playbook)

Experience log + decision playbook from the PMS Inventory System push failure — what happened, why, how it was fixed, and the reusable procedure for any future repo where pushing suddenly stops working.

---

## 📉 What Happened

- **Symptom:** `git push` on the local PMS repo (`C:\xampp\htdocs\PMS`) failed with:
  ```
  remote: Permission to Zurin2/PMS.git denied to Zurin28.
  fatal: unable to access 'https://github.com/Zurin2/PMS.git/': The requested URL returned error: 403
  ```
- **Commit state:** `main` was 1 commit ahead of `origin/main` (a full system rebuild commit) with a clean working tree — the commit was fine; it just could not leave the machine.
- **Duration:** the wrong credential had been sitting in Windows Credential Manager for a long time; it only became visible the first time a push was actually attempted.

## 🔍 Root Cause

Windows **Credential Manager** had a stored generic git credential **`git:https://Zurin28@github.com`** (an older/second GitHub account). When Git pushed over HTTPS, Git Credential Manager matched that generic `github.com` credential first and authenticated as **`Zurin28`** — an account with no write access to the **`Zurin2/PMS`** repository. GitHub correctly answered **403 Permission denied**.

A separate credential for `Zurin2` existed (`GitHub - https://api.github.com/Zurin2`), but the GitHub CLI (`gh`) was **not** logged in, so no helper route existed that keyed pushes to the right account.

## 🛠️ The Fix (Applied)

Surgical — the wrong-account credential was **left intact** (it may serve other repos):

1. Re-scoped the remote URL to embed the correct username:
   ```powershell
   git remote set-url origin https://Zurin2@github.com/Zurin2/PMS.git
   ```
   This makes Git Credential Manager look for a credential tied to `Zurin2@github.com` specifically, instead of falling back to the generic `github.com` entry owned by `Zurin28`.
2. Pushed: `d991201..353fa8c main -> main` ✅ (all future pushes on this repo now authenticate as `Zurin2`).
3. Verified alignment: `git status -sb` → `## main...origin/main` (in sync).

> ⚠️ PowerShell gotcha (noted during the incident): git writes push progress to **stderr**, which PowerShell displays as a red ".RemoteException" block — that does **not** mean failure. A successful push shows the ref update line (`old..new branch -> branch`) inside that output. Don't panic at red text; read the ref line.

---

## 🧰 Reusable Playbook — "Push to GitHub returns 403 / Permission denied to <someone>"

When a push fails and the error names a GitHub account that is **not yours**, work this checklist top-down:

1. **Confirm the identity mismatch first**
   ```powershell
   git status -sb          # confirm local commit state is fine
   git remote -v           # confirm the repo owner in the URL
   git push origin main    # read WHO GitHub says is denied
   ```
2. **List stored GitHub credentials**
   ```powershell
   cmdkey /list | Select-String -Context 0,3 "github|git"
   ```
   Look for `Target: ... git:https://<User>@github.com` entries — especially a generic one whose user ≠ repo owner.
3. **Choose the fix, most-surgical first**
   - ✅ **Per-repo scope (preferred):** set the remote to embed the correct username (see command in "The Fix"). Nothing global is touched; other repos keep their credentials.
   - 🧹 **Only if nothing else uses it:** delete the stale generic credential:
     ```powershell
     cmdkey /delete:"git:https://github.com"
     ```
     (Target name as shown by `cmdkey /list`; the next push re-prompts. **Never delete a login that other projects still push with.**)
   - 🧰 **If GitHub CLI is installed:** `gh auth login` as the correct account, then `gh auth setup-git` — this wires git's credential helper to the CLI and is the cleanest long-term routing.
4. **Credentials hygiene notes**
   - GitHub no longer accepts account passwords for git; pushes use **PATs or browser OAuth** via Git Credential Manager. Completing the popup browser flow once stores a working credential.
   - A PAT must have `repo` scope for private repos; expired/rotated PATs also surface as 403/401.
5. **Verify the fix**
   ```powershell
   git push origin main
   git status -sb         # "## main...origin/main" with no "ahead/behind" = synced
   ```
6. **If the account itself lacks access** (GitHub shows the correct username in the error): the repo owner must grant **Collaborator access** (Settings → Collaborators) — no local credential change can fix that.

## 🛡️ Prevention — Stop It Happening Again

- **Scope every new remote with the account that owns it:**
  ```powershell
  git remote add  origin https://Zurin2@github.com/Zurin2/<repo>.git
  # or retrofit existing:
  git remote set-url origin https://Zurin2@github.com/Zurin2/<repo>.git
  ```
- **Or make Git Credential Manager path-aware globally** so generic github.com credentials can't shadow project-specific ones:
  ```powershell
  git config --global credential.https://github.com.useHttpPath true
  ```
  (Each repo's credential is then stored under its full URL — recommended one-time setup.)
- **Own one identity per machine:** prefer `gh auth login` + `gh auth setup-git`, or GitHub Desktop, and avoid keeping multiple raw account credentials in Credential Manager; audit periodically with `cmdkey /list`.
- **After cloning on a new machine or from another person's machine:** run a throwaway `git ls-remote origin` — it authenticates cheaply and reveals a wrong-account 403 **before** your first real push.
- **Before pushing anything for the first time:** `git remote -v` + whoami check = 10 seconds that prevents this entire incident class.

---

## 🔗 Related Notes

* 📦 **[[PMS Inventory System - Project Architecture & Requirements Proposal]]** — the project this failing repo belongs to.
* 🗄️ **[[PMS Inventory System — Database Design & Implementation Plan]]** — codebase specification of the same system.
* 🚀 **[[Projects & Software Development (Map of Content)]]** — parent MOC.
* 💭 **[[Personal Knowledge Hub (Map of Content)]]** — developer tooling knowledge.
