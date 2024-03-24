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
cat ~/.ssh/unpluggedkk_id_rsa.pub
```

## Step 6: Bootstrap FluxCD

```sh
flux bootstrap git --url=ssh://git@github.com/unplugged-kk/fluxcd-crossplane-idp \
--branch=main --private-key-file=/Users/kishore.behera/unpluggedkk_id_rsa --password=<SSH-PRIVATE-KEY-PASSWORD> \
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