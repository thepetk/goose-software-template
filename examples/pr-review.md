# Use Case: Review a GitHub Pull Request

Deploy a template instance configured with your LLM provider, then create a recipe that instructs the agent to review a specific PR and post its findings.

## Prerequisites

- A running Goose agent pod (deployed via this template)
- A GitHub personal access token with `repo` scope

## Recipe

Create `pr-review.yaml`:

```yaml
version: 1.0.0
title: GitHub PR Review
description: Review a GitHub Pull Request and post actionable feedback
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
  You are a thorough code reviewer. Review the provided GitHub pull request,
  check for bugs, security issues, code quality, and adherence to best
  practices. Post your findings as review comments directly on the PR.
prompt: "Review PR #42 in myorg/myrepo. Leave inline comments for any issues found."
```

## Running

Copy the recipe into the pod and run it:

```bash
kubectl cp pr-review.yaml <namespace>/<pod-name>:/tmp/pr-review.yaml

kubectl exec -it <pod-name> -n <namespace> -- \
  env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
  goose run --recipe /tmp/pr-review.yaml
```

The agent will open the PR diff via the GitHub MCP server, analyse it, and post review comments back to GitHub.
