# cluster-apps

ArgoCD apps-of-apps repository for managing cluster workloads.

## One-time bootstrap steps

Some secrets cannot be stored in git and must be applied manually before ArgoCD syncs the relevant applications.

### External Secrets + 1Password Connect

Before pushing the external-secrets changes, complete the following in [1password.com](https://1password.com):

1. Create a vault (e.g. `k8s-secrets`)
2. Go to **Developer Tools → Directory → 1Password Connect**
3. Click **New Integration**, select your vault
4. Download `1password-credentials.json`
5. Click **Issue Token**, copy the token (shown only once)

Then apply the bootstrap secrets to your cluster:

```bash
kubectl create namespace external-secrets

# Credentials file for the Connect server pod.
# NOTE: must use --from-literal with base64-encoded content, NOT --from-file.
# The Connect server expects to base64-decode this value itself.
kubectl create secret generic onepassword-connect-credentials \
  --from-literal=1password-credentials.json="$(cat /path/to/1password-credentials.json | base64 -w 0)" \
  -n external-secrets

# Access token for ESO to call the Connect server
kubectl create secret generic onepassword-connect-token \
  --from-literal=token=<your-access-token> \
  -n external-secrets
```

After these secrets exist, ArgoCD will successfully sync the `onepassword-connect` and `external-secrets` applications and the `ClusterSecretStore` will become ready.
