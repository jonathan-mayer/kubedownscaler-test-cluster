# kubedownscaler-test-cluster

A repo for holding kubernetes manifests and configurations for cd which are used for testing the kubedownscalers

## Setup with Kubernetes in Docker (KIND)

There are two versions available:

- kubedownscaler-test (has proxy configuration)
- kubedownscaler-test_np (without proxy configuration)

### Prerequisites and Creating the Cluster

```bash
brew install kind fluxcd/tap/flux

kind create cluster --name kubedownscaler-test --config kind/kubedownscaler-test/config.yaml
# to delete:
# kind delete cluster --name kubedownscaler-test
```

### GitHub PAT Setup

Go to https://github.com/settings/personal-access-tokens/new

Set repository access to the kubedownscaler-test-cluster repository.

Set the permissions to:

- Administration -> Access: Read-only
- Contents -> Access: Read and write
- Metadata -> Access: Read-only

Add this to your .env file.

### Setup FluxCD

```bash
dotenv
flux bootstrap github \
  --token-auth \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=flux/clusters/kubedownscaler-test \
  --personal
# to uninstall:
# flux uninstall
```
