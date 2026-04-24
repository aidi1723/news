# TrendRadar Telegram Daily Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Import TrendRadar into `aidi1723/news` and configure it to send one Telegram daily summary from GitHub Actions with minimal maintenance overhead.

**Architecture:** Keep the upstream codebase structure unchanged, run GitHub Actions every 2 hours for data collection, and rely on TrendRadar's scheduler to push only once during an evening summary window. Disable AI and RSS so the first version only depends on GitHub Actions plus Telegram secrets.

**Tech Stack:** GitHub Actions, Python 3.12, TrendRadar, YAML configuration, Telegram Bot API

---

### Task 1: Import upstream source into the target repository

**Files:**
- Create: upstream project files under repository root
- Preserve: `.git/`

- [ ] **Step 1: Verify the target repository is empty**

Run: `ls -la /Users/aidi/编程工具/news`
Expected: only `.git/` exists and there are no working files yet.

- [ ] **Step 2: Copy TrendRadar source into the target repository**

Run: `rsync -a --exclude '.git' /Users/aidi/编程工具/TrendRadar/ /Users/aidi/编程工具/news/`
Expected: the `news` repository now contains the TrendRadar source tree while keeping the `news` repository's own git metadata.

- [ ] **Step 3: Verify key files exist in the target repository**

Run: `ls -la /Users/aidi/编程工具/news`
Expected: `.github/`, `config/`, `trendradar/`, `README.md`, and `pyproject.toml` are present.

### Task 2: Reconfigure runtime behavior for Telegram daily summary only

**Files:**
- Modify: `config/config.yaml`
- Modify: `config/timeline.yaml`

- [ ] **Step 1: Update `config/config.yaml` for the minimal deployment**

Set:
- `schedule.preset` to `custom`
- `rss.enabled` to `false`
- `filter.method` to `keyword`
- `display.regions.ai_analysis` to `false`
- `ai_analysis.enabled` to `false`
- `ai_translation.enabled` to `false`

Expected result: the runtime no longer depends on AI services and only works with hot-topic platform data plus Telegram notification secrets.

- [ ] **Step 2: Update `config/timeline.yaml` custom preset**

Set the custom schedule to:
- collect all day by default
- never push by default
- define one evening summary period from `20:00` to `22:00`
- within that period, set `push: true`, `report_mode: "daily"`, and `once.push: true`

Expected result: GitHub Actions can run multiple times per day while Telegram only receives one evening summary.

### Task 3: Reconfigure GitHub Actions for long-term use

**Files:**
- Modify: `.github/workflows/crawler.yml`

- [ ] **Step 1: Remove the expiration check step**

Delete the workflow step named `Check Expiration`.
Expected result: the repository no longer disables itself after 7 days.

- [ ] **Step 2: Change cron scheduling to periodic collection**

Replace the default hourly cron with a 2-hour interval cron such as `17 */2 * * *`.
Expected result: the workflow wakes up often enough to accumulate daily data without burning unnecessary Actions minutes.

### Task 4: Add repository-level setup instructions

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add a short repository-specific setup section near the top**

Document:
- this repository is a self-hosted TrendRadar deployment
- required GitHub Secrets: `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`
- workflow behavior: collect every 2 hours, push one evening summary
- manual next step: enable Actions in GitHub if disabled

Expected result: the repository can be handed back to the user without hidden setup knowledge.

### Task 5: Verify configuration consistency

**Files:**
- Verify only

- [ ] **Step 1: Inspect modified files**

Run:
- `sed -n '1,220p' .github/workflows/crawler.yml`
- `sed -n '1,260p' config/config.yaml`
- `sed -n '397,520p' config/timeline.yaml`

Expected: workflow has no expiration step, config disables AI/RSS, timeline custom preset contains one evening summary push window.

- [ ] **Step 2: Review final git status**

Run: `git -C /Users/aidi/编程工具/news status --short`
Expected: imported files plus the planned config/docs changes are staged in working tree for review and push.
