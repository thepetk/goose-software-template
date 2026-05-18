# Use Case: License Compliance Check

When a pull request introduces new dependencies, the agent scans them against an allowlist and blocks merge (or opens an issue) if a disallowed license is detected.

## Prerequisites

- A running Goose agent pod (deployed via this template)
- A GitHub personal access token with `repo` scope
- An allowlist of approved licenses (e.g. MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause)

## Recipe

Create `license-compliance.yaml`:

```yaml
version: 1.0.0
title: License Compliance Check
description: Scan new dependencies in a PR for disallowed licenses
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
    timeout: 300
    bundled: true
instructions: |
  You are a license compliance assistant. Given a pull request:
  1. Identify any dependency manifest files changed (package.json, go.mod,
     requirements.txt, pom.xml, Gemfile, etc.).
  2. Extract newly added dependencies from the diff.
  3. For each new dependency, determine its declared license using available
     tools (npm info, pip show, go list, etc.) or by reading the repository.
  4. Check each license against the approved list:
       MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, MPL-2.0
  5. If any dependency carries a disallowed license (e.g. GPL, AGPL, SSPL,
     proprietary), post a review comment on the PR listing the offending
     packages and request changes.
  6. If all licenses are approved, post an approving comment confirming compliance.
prompt: "Run a license compliance check on PR #{{ pr_number }} in {{ repo }}."
```

## Running

### On demand

```bash
kubectl exec -it <pod-name> -n <namespace> -- \
  env GITHUB_PERSONAL_ACCESS_TOKEN=<token> \
  goose run --recipe /tmp/license-compliance.yaml \
    --params pr_number=42 repo=myorg/myrepo
```

### As a CI step (GitHub Actions)

```yaml
- name: License compliance check
  run: |
    kubectl exec -it $POD -n $NS -- \
      env GITHUB_PERSONAL_ACCESS_TOKEN=${{ secrets.GH_TOKEN }} \
      goose run --recipe /tmp/license-compliance.yaml \
        --params pr_number=${{ github.event.pull_request.number }} \
                 repo=${{ github.repository }}
```
