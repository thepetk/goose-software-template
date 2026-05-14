# Template Usage

## Running the Template

### Step 1: Goose Agent Configuration

| Field | Description |
|-------|-------------|
| **Name** | A unique identifier for this agent instance. Used as the Kubernetes resource name and ArgoCD app name (max 63 characters, lowercase, no spaces). |
| **Owner** | The Backstage catalog owner (user or group). |
| **LLM Provider** | The LLM backend Goose will use. Choose from Anthropic, OpenAI, Google, Ollama, or OpenRouter. |
| **Model Name** | The specific model identifier for your chosen provider (e.g. `claude-sonnet-4`, `gpt-4o`). Must match the provider's model identifier exactly. |
| **ArgoCD Namespace** | The namespace where ArgoCD is installed (usually `openshift-gitops`). |
| **ArgoCD Project** | The ArgoCD project to deploy into (usually `default`). |

### Step 2: Repository Information

| Field | Description |
|-------|-------------|
| **Host Type** | GitHub or GitLab — determines which publisher action is used. |
| **Repository Owner** | The GitHub organization/user or GitLab group/user that will own the repository. |
| **Repository Name** | The base name for the repository. The GitOps repo will be created as `<repoName>-gitops`. |
| **Default Branch** | The default branch name (usually `main`). |

### Step 3: Deployment Information

| Field | Description |
|-------|-------------|
| **Goose Container Image** | The Goose image to deploy. Defaults to `ghcr.io/block/goose:latest`. Pin to a specific tag for production (e.g. `ghcr.io/block/goose:v1.34.0`). |
| **Deployment Namespace** | The OpenShift namespace where Goose will be deployed. ArgoCD will create it if it does not exist. |
| **Deploy on Remote Cluster** | Enable to deploy to a different OpenShift cluster via ArgoCD. Requires the cluster to be registered in ArgoCD. |

## What Gets Created

After the template runs successfully:

1. **GitOps repository** (`<repoName>-gitops`) created in your GitHub/GitLab organization
2. **ArgoCD Application** registered, pointing to the new GitOps repository
3. **Backstage catalog entity** registered as a `Resource` of type `gitops`

ArgoCD will immediately begin synchronizing the manifests. The Goose deployment will remain in a pending state until you create the provider API key Secret:

```bash
kubectl create secret generic <name>-provider-api-key \
  --from-literal=<PROVIDER_API_KEY_ENV>=<your-api-key> \
  -n <namespace>
```

## Accessing the Goose Agent

Once deployed, the Goose agent's REST+SSE API is accessible via the OpenShift Route created in your namespace. You can retrieve the Route URL with:

```bash
oc get route <name> -n <namespace> -o jsonpath='{.spec.host}'
```

The Goose API runs on port 3000. Refer to the [Goose documentation](https://goose-docs.ai/) for the API reference.

## Customizing the Deployment

To modify the deployment after scaffolding, edit the Kubernetes manifests in the GitOps repository. ArgoCD will automatically apply changes on the next sync.

Common customizations:
- **Change the model**: Update `GOOSE_PROVIDER` and `GOOSE_MODEL` in `components/<name>/base/deployment.yaml`
- **Scale replicas**: Update `spec.replicas` in `components/<name>/overlays/development/deployment-patch.yaml`
- **Pin the image version**: Update the image tag in `deployment-patch.yaml`
- **Add MCP extensions**: Mount additional config via a ConfigMap and set `GOOSE_EXTENSIONS` env vars
