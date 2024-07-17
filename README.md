# Hashicorp-Vault-Secrets-in-Kubernetes-with-CSI-Driver
Injecting secrets to Kubernetes Pods using Vault CSI Driver.

Prerequisites: Kubernetes Cluster & Vault Server

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

3. Create ClusterRoleBinding
```
kubectl apply -f vaultCRB.yaml
```
4. Create a service account
```
kubectl apply -f vaultSA.yaml
```
5. Create a Secret for the service account
```
kubectl apply -f vaultSASecret.yaml
```
6. Export SA_JWT_TOKEN, SA_CA_CRT, K8S_HOST, VAULT_ADDR & VAULT_TOKEN   
```
export VAULT_ADDR=”<vault_address>”

export VAULT_TOKEN="<vault_token>"

export SA_JWT_TOKEN=$(kubectl get secret vault-sa-secret -n vault --output 'go-template={{ .data.token }}'| base64 --decode)

export SA_CA_CRT=$(kubectl config view --raw --minify --flatten --output 'jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)

export K8S_HOST=$(kubectl config view --raw --minify --flatten --output 'jsonpath={.clusters[].cluster.server}')

```

7. Create new Kubernetes authentication in Vault to establish a connection between Kubernetes and Vault.
```
vault auth enable --path=<cluster-name> kubernetes

vault write auth/<cluster-name>/config token_reviewer_jwt="$SA_JWT_TOKEN" kubernetes_host="$K8S_HOST" kubernetes_ca_cert="$SA_CA_CRT" issuer="https://kubernetes.default.svc.cluster.local"
```

8. Create a secret engine(KV Store) in vault and Add secrets to secret engine
```
vault secrets enable -path=<kv-store-name> kv

vault kv put <kv-store-name>/my-secret my-key=my-value
```

9. Create a Vault read policy for a specific path
```
vault policy write <policy-name>- <<EOF
path "<kv-store-name>/data/*" {
  capabilities = ["read"]
}
EOF

```

10. Create Vault Role
```
vault write auth/<cluster-name>/role/<role-name> \
bound_service_account_names=vault-sa \
bound_service_account_namespaces=* \
token_policies=<policy-name> 
```

11. Create a Secret Provider Class
```
kubectl apply -f vaultSPC.yaml
```
12. Add as volume in Deployment
```
kubectl apply -f vaultDeploy.yaml
```
    


   
