# kubedownscaler-test-cluster

A repo for holding kubernetes manifests and configurations for cd which are useful for testing the kubedownscalers

There are two versions available:

- kubedownscaler-test (has proxy configuration)
- kubedownscaler-test_np (without proxy configuration)

The downscaler itself is deployed via FluxCD.
FluxCD is deployed via bootstrap and ArgoCD is deployed via FluxCD.
Additionally, there will be some test applications deployed via FluxCD and ArgoCD. <!--TODO-->

## Prerequisites/Useful Kubernetes CLIs

```bash
brew install kubectl k9s helm
```

## Setup with Kubernetes in Docker (KIND)

### Install KIND CLI

```bash
brew install kind
```

### Creating the Cluster

```bash
kind create cluster --name kubedownscaler-test --config kind/kubedownscaler-test/config.yaml # or kind/kubedownscaler-test_np/config.yaml
# to delete:
# kind delete cluster --name kubedownscaler-test
```

## Setup FluxCD

### Install FluxCD CLI

```bash
brew install fluxcd/tap/flux
```

### GitHub PAT Setup

Go to https://github.com/settings/personal-access-tokens/new

Set repository access to the kubedownscaler-test-cluster repository.

Set the permissions to:

- Administration -> Access: Read-only
- Contents -> Access: Read and write
- Metadata -> Access: Read-only

Add this to your .env file.

### Install FluxCD on Cluster

```bash
dotenv
flux bootstrap github \
  --token-auth \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=flux/clusters/kubedownscaler-test \
  --personal # or --path=flux/clusters/kubedownscaler-test_np
# to uninstall:
# flux uninstall
```

## Setup ArgoCD

### Install ArgoCD CLI

```bash
brew install argocd
```

### Install ArgoCD on Cluster

Normally ArgoCD is installed via FluxCD so this is only required if you do not want to install Flux.
The Downscaler itself is also managed via flux so it will also have to be installed manually without Flux.

https://argo-cd.readthedocs.io/en/stable/operator-manual/installation

### Forward Argo Server

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Get Initial Password

```bash
argocd admin initial-password -n argocd
```

### Login to WebGui

Go to localhost:8080 and log in.w

### Login via Cli

```bash
argocd login localhost:8080
```

## Future Ideas

Make a helm chart that through the values.yaml file can start some deployments that are useful for testing the downscaler live.

values.yaml could look like this:

```yaml
deployments:
  replicas: 3
  image: nginx

  tests:
    - name: excluded
      annotations:
        "downscaler/excluded": "true"
    - name: included
      annotations:
        "downscaler/excluded": "false"
    - name: something-else
      annotations:
        "downscaler/an-option": "value"
        "downscaler/another-option": "another value"
```
