# Goose Software Template

A Backstage Software Template that deploys a [Goose](https://goose-docs.ai/) AI agent instance on OpenShift via GitOps.

## Overview

[Goose](https://goose-docs.ai/) is an open-source AI agent developed by Block (now under the [AAIF](https://aaif.io/) at the Linux Foundation). It autonomously executes complex developer workflows (running commands, debugging errors, writing and reading files, and orchestrating multi-step tasks) directly on your infrastructure.

This template scaffolds a GitOps repository that deploys a configured Goose agent instance on OpenShift using ArgoCD and Kustomize. No source code build is required; Goose just ships as a ready-to-run container image.

## Prerequisites

- Red Hat Developer Hub (RHDH) or Backstage with the scaffolder plugin
- OpenShift GitOps (ArgoCD) installed and configured in RHDH
- GitHub or GitLab integration configured in RHDH
- An API key for your chosen LLM provider (Anthropic, OpenAI, Google, etc.)

### Prepare Secrets

Create the provider API key Secret in your target namespace before the agent can connect to the LLM:

```bash
kubectl create secret generic <name>-provider-api-key \
  --from-literal=<PROVIDER_API_KEY_ENV>=<your-api-key> \
  -n <namespace>
```

For Anthropic: `ANTHROPIC_API_KEY`. For OpenAI: `OPENAI_API_KEY`. For Google: `GOOGLE_API_KEY`.

## Usage

### Import the Template

Add the following location to your Backstage `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/thepetk/goose-software-template/blob/main/template.yaml
```

Or import it directly via the RHDH catalog import UI.

### Run the Template

1. Navigate to **Create** in RHDH and find **Goose AI Agent**
2. Fill in the three parameter pages:
   - **Goose Agent Configuration** — name, LLM provider, model, ArgoCD settings
   - **Repository Information** — GitHub/GitLab org and repo name
   - **Deployment Information** — container image, namespace, optional remote cluster
3. Click **Create** — the template will scaffold a GitOps repository and register it in ArgoCD

## Template Parameters

| Parameter       | Description                                                            | Default                      |
| --------------- | ---------------------------------------------------------------------- | ---------------------------- |
| `name`          | Unique name for the agent instance (max 63 chars)                      | —                            |
| `owner`         | Backstage catalog owner                                                | `user:guest`                 |
| `gooseProvider` | LLM provider (`anthropic`, `openai`, `google`, `ollama`, `openrouter`) | `anthropic`                  |
| `gooseModel`    | Model name (e.g. `claude-sonnet-4`, `gpt-4o`)                          | `claude-sonnet-4`            |
| `argoNS`        | ArgoCD namespace                                                       | `openshift-gitops`           |
| `argoProject`   | ArgoCD project                                                         | `default`                    |
| `hostType`      | Git host (`GitHub` or `GitLab`)                                        | `GitHub`                     |
| `repoOwner`     | GitHub/GitLab org or user                                              | —                            |
| `repoName`      | Repository name (gitops repo will be `<repoName>-gitops`)              | —                            |
| `gooseImage`    | Goose container image                                                  | `ghcr.io/block/goose:latest` |
| `namespace`     | Deployment namespace                                                   | `goose-agent`                |
| `otlpEndpoint`  | OTLP collector endpoint for traces (e.g. `http://otel-collector:4318`) | `""` (disabled)              |

ArgoCD manages the full lifecycle: namespace creation, resource deployment, and continuous reconciliation.

## Examples

The [`examples/`](examples/) directory contains end-to-end walkthroughs for common use cases:

| Example | Description |
| ------- | ----------- |
| [PR Review](examples/pr-review.md) | Instruct the agent to review a specific GitHub pull request and post inline comments |
| [PR Description Enricher](examples/pr-description-enricher.md) | Automatically fill in a structured PR description when one is missing or sparse |
| [License Compliance Check](examples/license-compliance-check.md) | Scan new dependencies in a PR against an approved license allowlist |
| [Dependency Patrol](examples/dependency-patrol.md) | Scheduled daily job that bumps outdated packages and opens a draft PR per repository |
| [Scheduled Daily Job](examples/scheduled-job.md) | Register a cron-based recipe that runs autonomously every day |

## Contributing

Contributions are welcome. Please open an issue before submitting a pull request for significant changes.

## License

Apache License 2.0: see [LICENSE](LICENSE).
