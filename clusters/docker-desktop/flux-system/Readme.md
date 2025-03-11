# FluxCD Installation Guide for Docker Desktop

This guide is the installation of FluxCD on Docker Desktop . FluxCD is a tool for continuous delivery and GitOps, enabling automated deployments to Kubernetes based on Git repository changes.

## Prerequisites

- Flux CLI
- Helm
- Git
- GitHub CLI (optional, for FluxCD repository integration)



## Step 1: Install Flux CLI

Install FLux CLI on Mangement Server.

```sh
curl -s https://fluxcd.io/install.sh | sudo bash
```

```sh
kubectl create ns flux-system
```


## Step 2: Install FluxCD Helm Chart

```sh
helm install -n flux-system flux oci://ghcr.io/fluxcd-community/charts/flux2
```

## Step 3: Install Required Packages

```sh
sudo yum install git
sudo dnf install 'dnf-command(config-manager)'
sudo dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
sudo dnf install gh
```

## Step 4: Generate SSH Key

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

## Step 5: Display Public Key

```sh
cat /Users/kishore.behera/.ssh/kishore.pub
```

## Step 6: Bootstrap FluxCD

```sh
flux bootstrap git --url=ssh://git@github.com/kishore-behera/fluxcd-crossplane-idp \
--branch=main --private-key-file=/Users/kishore.behera/.ssh/kishore --password=<SSH-PRIVATE-KEY-PASSWORD> \
--path=clusters/docker-desktop  < you can use your cluster name too >
```

## Verify Installation

Check if FluxCD has been successfully installed by listing Helm releases:

```sh
helm list --all-namespaces
```

## Additional Notes

- Ensure that the SSH key generated in Step 4 is added as a deploy key with write access to Git repository.
- Adjust configurations such as repository URL, branch, and path according to project structure.
- Keep your private key secure and do not share it with unauthorized users.

# Komoplane 
```sh
helm repo add komodorio https://helm-charts.komodor.io \
  && helm repo update komodorio \
  && helm upgrade --install komoplane komodorio/komoplane
```  

# Helm Dashboard

https://github.com/komodorio/helm-dashboard

```sh
helm plugin install https://github.com/komodorio/helm-dashboard.git && helm plugin update dashboard

```
To run use 

```sh
helm dashboard
```


##  Sealed Secret 

### 1. Add Sealed Secrets Helm Repository

```sh
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
```

### 2. Install Sealed Secrets

```sh
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
```

### 3. Install Kubeseal CLI

```sh
KUBESEAL_VERSION='0.26.0' wget "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz" tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

### 4. Create Sealed Secret

Create a `azure-credentials.json` file with your Azure credentials and then seal it:

```json
{
  "clientId": "xxxxx",
  "clientSecret": "xxxxx",
  "tenantId": "xxxxx",
  "subscriptionId": "xxxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

```sh
bash kubectl create secret generic azure-secret
-n crossplane-system
--from-file=creds=./azure-credentials.json
--dry-run=client -o yaml | kubeseal --format=yaml > sealed-azure-secret.yaml
```