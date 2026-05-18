# Use Case: PR Description Enricher

When a pull request is opened with a sparse or missing description, the agent reads the diff and fills in a structured template covering summary, motivation, test plan, and risk.

## Prerequisites

- A running Goose agent pod (deployed via this template)
- A GitHub personal access token with `repo` scope
- A GitHub webhook or CI step that triggers the job on `pull_request.opened` events

## Recipe

Create `pr-description-enricher.yaml`:

```yaml
version: 1.0.0
title: PR Description Enricher
description: Read a PR diff and write a structured description if one is missing
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
  You are a developer-experience assistant. When given a GitHub pull request:
  1. Fetch the PR metadata and diff.
  2. If the description is already detailed (more than 100 words), do nothing.
  3. Otherwise, update the PR description with the following sections:
     - **Summary**: what changed and why, in 2-3 sentences.
     - **Changes**: a bullet list of the key modifications.
     - **Test plan**: how the change was or should be tested.
     - **Risk**: any breaking changes, migrations, or rollback considerations.
  Keep the tone concise and factual. Do not invent information not present in the diff.
prompt: "Enrich the description of PR #{{ pr_number }} in {{ repo }} if it is sparse."
```

## Running

### On demand

```bash
kubectl exec -it <pod-name> -n <namespace> -- \
  env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
  goose run --recipe /tmp/pr-description-enricher.yaml \
    --params pr_number=42 repo=myorg/myrepo
```

### As a CI step (GitHub Actions)

```yaml
- name: Enrich PR description
  run: |
    kubectl exec -it $POD -n $NS -- \
      env GITHUB_PERSONAL_ACCESS_TOKEN=${{ secrets.GH_TOKEN }} \
      goose run --recipe /tmp/pr-description-enricher.yaml \
        --params pr_number=${{ github.event.pull_request.number }} \
                 repo=${{ github.repository }}
```
