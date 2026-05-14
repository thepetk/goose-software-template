# ${{ values.name }} — Goose Agent GitOps

This repository manages the GitOps configuration for a [Goose](https://goose-docs.ai/) AI agent deployment on OpenShift.

## Deployed Resources

| Resource   | Description                                                        |
| ---------- | ------------------------------------------------------------------ |
| Deployment | Goose agent running `${{ values.gooseImage }}`                     |
| Service    | ClusterIP service exposing port 3000                               |
| Route      | OpenShift Route with edge TLS termination                          |
| ConfigMap  | `${{ values.name }}-goose-config` with provider and model settings |

## Configuration

The Goose agent is configured for:

- **Provider**: `${{ values.gooseProvider }}`
- **Model**: `${{ values.gooseModel }}`

The agent connects to the LLM provider using an API key stored in the `${{ values.name }}-provider-api-key` Secret in the `${{ values.namespace }}` namespace. This Secret must be created manually before the agent can communicate with the LLM provider.
