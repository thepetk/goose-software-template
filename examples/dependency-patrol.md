# Use Case: Dependency Update Patrol

A scheduled daily job that scans repositories for outdated packages and opens a draft pull request per repo with the version bumps applied.

## Prerequisites

- A running Goose agent pod (deployed via this template)
- A GitHub personal access token with `repo` scope and permission to open PRs

## Recipe

Create `dependency-patrol.yaml`:

```yaml
version: 1.0.0
title: Dependency Update Patrol
description: Scan for outdated dependencies and open a draft PR with version bumps
extensions:
  - type: stdio
    name: github
    cmd: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env_keys:
      - GITHUB_PERSONAL_ACCESS_TOKEN
    timeout: 300
    bundled: false
  - type: builtin
    name: developer
    timeout: 600
    bundled: true
instructions: |
  You are a dependency maintenance assistant. For the configured repository:
  1. Clone or fetch the default branch.
  2. Detect the package ecosystem (npm, pip, Go modules, Maven, etc.) from
     the manifest files present.
  3. Run the appropriate tool to list outdated packages:
       npm outdated, pip list --outdated, go list -u -m all, etc.
  4. Apply patch and minor version updates only (skip major version bumps to
     avoid breaking changes).
  5. Create a new branch named deps/auto-update-<date>.
  6. Commit the updated manifest and lockfile with the message
     "chore: auto-update dependencies <date>".
  7. Open a draft pull request titled "chore: dependency updates <date>"
     with a table listing each package, old version, and new version.
  Do not update packages that have no newer version. Skip pre-release versions.
prompt: "Run the dependency patrol for {{ repo }}."
```

## Registering the Schedule

Copy the recipe into the pod and register a daily job:

```bash
kubectl cp dependency-patrol.yaml <namespace>/<pod-name>:/tmp/dependency-patrol.yaml

# Runs every day at 07:00 UTC
kubectl exec -it <pod-name> -n <namespace> -- \
  env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
  goose schedule add \
    --schedule-id dependency-patrol \
    --cron "0 7 * * *" \
    --recipe-source /tmp/dependency-patrol.yaml
```

Verify the schedule:

```bash
kubectl exec -it <pod-name> -n <namespace> -- goose schedule list
```

## Scanning multiple repositories

Run the recipe once per repository by passing a different `repo` param:

```bash
for REPO in myorg/service-a myorg/service-b myorg/service-c; do
  kubectl exec -it <pod-name> -n <namespace> -- \
    env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
    goose run --recipe /tmp/dependency-patrol.yaml \
      --params repo=$REPO
done
```

## Persistence Note

The pod's `emptyDir` volume keeps schedule state within the same pod lifetime. For durability across pod replacements, mount a `PersistentVolumeClaim` at `/home/goose/.local` instead of the default `emptyDir`.
