# TrendRadar Telegram Daily Deployment Design

## Goal

Deploy TrendRadar into the `aidi1723/news` repository with the smallest practical set of changes so it can collect hot-topic data throughout the day and send one Telegram daily summary through GitHub Actions.

## Scope

This design keeps TrendRadar's application code and structure intact and only changes:

- repository contents in `aidi1723/news`
- GitHub Actions scheduling and expiration behavior
- runtime configuration for daily summary behavior
- repository documentation for Telegram secrets and setup

This design does not add new product features, custom parsers, custom APIs, or AI-based analysis.

## Constraints

- Use the user's existing repository: `https://github.com/aidi1723/news`
- Keep changes minimal and easy to maintain
- Avoid AI model dependencies and related cost/risk for the first version
- Use Telegram as the only delivery channel
- Favor stable daily summary behavior over realtime alerts

## Key Decisions

### 1. Import the upstream source into `news`

The repository is currently empty, so the cleanest path is to copy the TrendRadar source tree into `news` while preserving the `news` repository's own `.git` history.

### 2. Remove the 7-day expiration gate

The upstream GitHub Actions workflow includes a trial-style expiration check. In the user's own repository this provides no value and would cause avoidable service interruptions, so it should be removed.

### 3. Do not run only once per day

A real daily summary needs data collected during the day. Running the crawler only once at night would produce a snapshot, not an all-day summary.

The workflow should therefore:

- run periodically during the day
- collect data every run
- push only once inside a configured evening window

### 4. Use a custom schedule preset

The safest minimal behavior is:

- collect all day
- do not push during the day
- push one `daily` summary in the evening

This will be implemented through `schedule.preset: "custom"` plus a custom evening summary period in `config/timeline.yaml`.

### 5. Disable AI and RSS in the first version

Upstream defaults enable AI-related behavior and RSS feeds. For the user's stated goal of basic hot-topic daily pushes:

- AI should be disabled to avoid API requirements and cost
- RSS should be disabled to avoid unrelated feed noise
- keyword filtering should be used instead of AI filtering

## Assumptions

- Default daily push window: Beijing time `20:00-22:00`
- GitHub Actions collection frequency: once every 2 hours
- Telegram secrets will be configured in GitHub repository settings after code is pushed

## Files Expected To Change

- `.github/workflows/crawler.yml`
- `config/config.yaml`
- `config/timeline.yaml`
- `README.md`

## Success Criteria

- The `news` repository contains a working TrendRadar codebase
- GitHub Actions no longer stops after 7 days
- The workflow collects data regularly without daytime Telegram pushes
- Telegram receives one evening summary each day after secrets are configured
- The repository README tells the user exactly which secrets to add and how to enable the workflow
