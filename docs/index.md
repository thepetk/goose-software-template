# Goose Software Template

Deploy a [Goose](https://goose-docs.ai/) AI agent instance on OpenShift via GitOps.

## What is Goose?

[Goose](https://goose-docs.ai/) is an open-source AI agent developed by Block (now hosted at the [Agentic AI Foundation](https://aaif.io/) under the Linux Foundation). Unlike code-completion tools, Goose autonomously executes complex developer workflows — it runs shell commands, reads and writes files, debugs errors, orchestrates multi-step tasks, and connects to external services via the Model Context Protocol (MCP).

Goose supports 15+ LLM providers:

| Provider | Example Models |
|----------|---------------|
| Anthropic | `claude-sonnet-4`, `claude-opus-4-7` |
| OpenAI | `gpt-4o`, `o3` |
| Google | `gemini-2.5-pro`, `gemini-2.5-flash` |
| Ollama | Any locally hosted model |
| OpenRouter | Any model via OpenRouter |

## What This Template Creates

This template scaffolds a **GitOps repository** that deploys a configured Goose agent instance on OpenShift using ArgoCD and Kustomize. No source code compilation is required — Goose ships as a ready-to-run container image (`ghcr.io/block/goose:latest`).

The deployed architecture includes:

- A **Deployment** running the `goosed agent` server (REST+SSE API on port 3000)
- A **Service** exposing the agent within the cluster
- An **OpenShift Route** for external access with edge TLS termination
- A **ConfigMap** storing the provider and model configuration

## How It Works

1. You fill in the template parameters (agent name, LLM provider, model, target namespace, git repository details)
2. The scaffolder creates a GitOps repository with all Kubernetes manifests
3. ArgoCD detects the new repository and synchronizes the resources to your cluster
4. You create a Kubernetes Secret with your LLM provider API key
5. The Goose agent starts and is accessible via the OpenShift Route

See [Prerequisites](prerequisites.md) and [Template Usage](usage.md) for step-by-step instructions.
