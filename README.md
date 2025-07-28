# 🛡️ GitGuardian Secret Scanner & Cleanup

Automated secret scanning and cleanup tools for your GitHub repository using GitGuardian’s `ggshield`.  
Includes a **scheduled secret scan workflow** and a **manual cleanup workflow** with optional history rewriting and github artifact export to get a rewritten repo.

image

---

## 🔍 Functions

- ✅ Auto-scan commits and pull requests for secrets using GitGuardian  
- 🗓️ Possibility to apply cron if wanted  
- 🧼 Manual workflow to **clean secrets from history** using [`git-filter-repo`](https://github.com/newren/git-filter-repo)
- 🧪 Dry-run mode for safe testing  
- 📦 Option to download cleaned repositories as artifacts  
- 🧹 Optional cleanup of GitHub Actions workflow runs

---

## 🚀 Secret Scan (Auto)

The `main.yml` workflow runs:

- On every push to `main`
- Every Monday at `00:00 UTC` or if cron is changed

It uses `ggshield` to scan the full repo for secrets using your `GITGUARDIAN_API_KEY`.

```yaml
env:
  GITGUARDIAN_API_KEY: ${{ secrets.GITGUARDIAN_API_KEY }}
```

### Setup

1. Create a GitGuardian API key [here](https://dashboard.gitguardian.com/settings/api/personal-access-tokens?id=true)
2. Add it to your repository secrets as `GITGUARDIAN_API_KEY`
3. Make sure it has permissions like scan and maybe incidents 
4. The scan now works

---

## 🧼 Cleanup Workflow (Manual)

The `cleanup-secrets.yml` workflow lets you manually:

- Scan for committed secrets  
- Replace one or more exposed secrets  
- (Optionally) Rewrite history and clean the full repo  
- Download cleaned versions as artifacts  
- Delete older GitHub Actions/workflow runs if needed, only doesn't remove itself.

You can trigger this workflow manually via the **Actions** tab → _"Cleanup Secrets (Manual Trigger)"_.

---

## ⚙️ Inputs for Cleanup

| Input                  | Description                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| `confirm`             | Type `"yes"` to confirm rewriting history                                    |
| `dry_run`             | `"true"` (default) will only show findings without cleanup                  |
| `secrets_to_replace`  | Comma-separated list of secrets (e.g. `abc123,ghp_456,apikey_xyz`)           |
| `save_cleaned_repo`   | `"true"` to download cleaned repo artifacts after the run                   |
| `delete_all_runs`     | `"true"` will attempt to delete all GitHub Actions workflow runs           |

---

## 🧾 Outputs & Artifacts

If you set `save_cleaned_repo: "true"` and confirmed cleanup, you'll get two folders:

- `cleaned-repo-bare.tar` → bare Git repository  
- `cleaned-repo-normal.tar` → normal working directory clone  

You’ll find them as **artifacts** attached to the workflow run.
Make sure you have something like [7-zip](https://www.7-zip.org/) to unzip the tar or tar.gz

---

## 🧠 How to Use the Cleaned Repo

After downloading, you can push the cleaned history using one of these methods:

### 🔁 Option A – Replace with working directory

```bash
tar -xzf cleaned-repo-normal.tar.gz
cd cleaned-repo-normal
git remote set-url origin https://github.com/your/repo.git
git push --force --all
git push --force --tags
```

### 🧱 Option B – Replace with bare repository

```bash
tar -xzf cleaned-repo-bare.tar.gz
cd repo.git
git remote add origin https://github.com/your/repo.git
git push --mirror --force
```

> ⚠️ This will **overwrite the entire repository history**. You could use the same github repo but you could also use another one first.

---

## 📦 Requirements

- GitGuardian API key (`GITGUARDIAN_API_KEY`)  
- GitHub Actions correctly set up with permissions for read & write
- git, if you want to push the rewritten history of a repository

---
