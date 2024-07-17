# Hashicorp-Vault-Secrets-in-Kubernetes-with-CSI-Driver
Injecting secrets to Kubernetes Pods using Vault CSI Driver


1. Install secrets-store-csi-driver & vault-csi-provider

```
helm repo add hashicorp https://helm.releases.hashicorp.com

helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts

helm install csi secrets-store-csi-driver/secrets-store-csi-driver --set syncSecret.enabled=true -n vault --create-namespace


helm install vault hashicorp/vault   --set "server.enabled=false"   --set "injector.enabled=false"   --set "csi.enabled=true" -n vault
```

2. Edit the daemon set and add toleration. This will match any node that has a taint.

```
kubectl edit ds vault-csi-provider -n vault

tolerations:
- operator: Exists
```
