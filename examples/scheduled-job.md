# Use Case: Create a Scheduled Daily Job

Deploy a template instance and register a cron-based scheduled job. The agent will execute the recipe autonomously on the defined schedule without further intervention.

## Prerequisites

- A running Goose agent pod (deployed via this template)
- A GitHub personal access token with `repo` scope

## Recipe

Create `daily-report.yaml`:

```yaml
version: 1.0.0
title: Daily Standup Report
description: Summarise overnight repository activity and open incidents
extensions:
  - type: stdio
    name: github
    cmd: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env_keys:
      - GITHUB_PERSONAL_ACCESS_TOKEN
    timeout: 300
    bundled: false
instructions: |
  You are a daily reporting assistant. Each morning, collect all pull requests
  merged and issues closed in the last 24 hours for the configured repository,
  then write a concise standup summary and open a new GitHub issue titled
  "Daily Summary – <date>" with your findings.
prompt: "Generate today's standup summary for myorg/myrepo."
```

## Registering the Schedule

Copy the recipe into the pod and register the job:

```bash
kubectl cp daily-report.yaml <namespace>/<pod-name>:/tmp/daily-report.yaml

# Runs every day at 09:00 UTC
kubectl exec -it <pod-name> -n <namespace> -- \
  env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
  goose schedule add \
    --schedule-id daily-standup \
    --cron "0 9 * * *" \
    --recipe-source /tmp/daily-report.yaml
```

Verify the schedule was registered:

```bash
kubectl exec -it <pod-name> -n <namespace> -- goose schedule list
```

## Persistence Note

The pod's `emptyDir` volume keeps schedule state across container restarts within the same pod lifetime. For durability across pod replacements, mount a `PersistentVolumeClaim` at `/home/goose/.local` instead of the default `emptyDir`.
