# Prerequisites

Before using this template, ensure the following are in place.

## Platform Requirements

- **OpenShift cluster** with cluster-admin or sufficient namespace permissions
- **OpenShift GitOps** (ArgoCD) installed — available via OperatorHub as the "Red Hat OpenShift GitOps" operator
- **Red Hat Developer Hub (RHDH)** or upstream Backstage with the following plugins enabled:
  - Backstage Scaffolder (built-in)
  - ArgoCD plugin (`@roadiehq/backstage-plugin-argo-cd-backend`)
  - GitHub or GitLab integration

## RHDH Configuration

Ensure your RHDH instance has:

1. **GitHub or GitLab integration** — OAuth app or token configured so the scaffolder can create repositories
2. **ArgoCD integration** — ArgoCD backend plugin connected to your ArgoCD instance
3. **This template registered** — Add the template URL to your `catalog` locations in `app-config.yaml`

## LLM Provider API Key

You need an API key for your chosen LLM provider:

| Provider | Environment Variable | Where to Get It |
|----------|---------------------|-----------------|
| Anthropic | `ANTHROPIC_API_KEY` | [console.anthropic.com](https://console.anthropic.com) |
| OpenAI | `OPENAI_API_KEY` | [platform.openai.com](https://platform.openai.com) |
| Google | `GOOGLE_API_KEY` | [aistudio.google.com](https://aistudio.google.com) |
| OpenRouter | `OPENROUTER_API_KEY` | [openrouter.ai](https://openrouter.ai) |
| Ollama | _(no key required)_ | Self-hosted |

After the template runs, you must create a Kubernetes Secret in the target namespace:

```bash
kubectl create secret generic <name>-provider-api-key \
  --from-literal=ANTHROPIC_API_KEY=<your-key> \
  -n <namespace>
```

Replace `ANTHROPIC_API_KEY` with the correct environment variable for your provider.
