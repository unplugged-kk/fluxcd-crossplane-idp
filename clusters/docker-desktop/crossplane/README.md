## Crossplane Installation with Flux CD

Welcome to the installation guide for Crossplane using Flux CD on your Private Kubernetes cluster for MDP Backbone.

## Prerequisites

To successfully install Crossplane using Flux CD, ensure you have the following prerequisites:

- A Kubernetes cluster
- Flux CD installed in your cluster
- Helm installed
- kubectl installed
- Sealed Secrets installed

## Installation Steps

Follow these steps to install Crossplane with Flux CD:

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

### 5. Apply Kustomization / Install Crossplane via Flux CD

Create the Kustomization structure

```sh
mkdir -p clusters/development/cloud-backbone-dev-aks-cluster1/crossplane
```

Apply the kustomization for Crossplane and related resources:

```sh
kubectl apply -k .
```

### 6. Verify Installation

Check the Crossplane installation:

```sh
kubectl get all -n crossplane-system
```

# Crossplane Composite Resources Overview

Explore the world of Crossplane Composite Resources with this comprehensive guide. Discover their significance, interrelations, and practical workflow examples for efficient infrastructure management.

## Key Concepts

### Compositions

Compositions are essential in Crossplane for crafting opinionated infrastructure APIs. They empower platform teams to develop reusable templates that encapsulate best practices, standards, and guardrails for provisioning and managing infrastructure across various cloud providers.

### Composite Resource Definition (XRD)

XRDs define the schema for custom resource types in Crossplane known as Composite Resources (XRs). They specify the API group, version, kind, and configurable fields, enabling platform teams to expose relevant configuration options to application teams while abstracting underlying managed resources.

### Composite Resource (XR)

XR represents the desired state and configuration for infrastructure defined by application teams. Leveraging associated Compositions, Crossplane orchestrates the provisioning of necessary managed resources, offering a declarative approach for application teams to manage complex infrastructure without delving into cloud provider specifics.

### Claims (XRC)

Composite Resource Claims (XRCs) facilitate namespace-scoped provisioning and management of XRs. They act as proxies, enabling application teams to interact with infrastructure using familiar Kubernetes constructs while enforcing RBAC and isolation policies.

### Managed Resources

Managed Resources are the actual resources provided by cloud providers, such as Azure VMs and AKS clusters, provisioned and managed by Crossplane based on XR configurations. Compositions map and configure these resources, allowing platform teams to enforce best practices and standards through reusable infrastructure templates.

## End-to-End Workflow Example

Follow this example workflow for provisioning an AKS cluster using Crossplane:

1. Define an XRD for an "AKSCluster" resource with configurable fields.
2. Create a Composition mapping "AKSCluster" XR to essential Azure resources (e.g., Resource Group, VNet, AKS, ACR, Key Vault).
3. Generate an "AKSCluster" XRC within your namespace, configuring it as desired.
4. Crossplane provisions Azure resources corresponding to the AKS XR based on the Composition.
5. Access AKS cluster credentials via kubectl to deploy workloads.
6. Crossplane ensures provisioned resources match the desired state defined in the AKS XR.
7. Upon deletion of the AKSCluster XRC, Crossplane deprovisions related Azure resources.

![Crossplane Workflow Diagram](./images/crossplane_workflow_diagram.png)

## Configuration Files

```yaml
- `crossplane-helmrelease.yaml`: HelmRelease for Crossplane.
- `crossplane-helmrepository.yaml`: HelmRepository for Crossplane charts.
- `sealed-azure-secret.yaml`: SealedSecret for Azure credentials.
- `provider-config.yaml`: ProviderConfig for Azure provider.
- `provider-azure-network.yaml`: Provider for Azure network resources.
- `provider-azure-management.yaml`: Provider for Azure management resources.
- `crossplane-kustomization.yaml`: Kustomization for Crossplane managed by Flux CD.
- `nlz-kustomization.yaml`: Kustomization for Composite Resources created via Crossplane managed by Flux CD.
- `xrds-kustomization.yaml`: Kustomization for Composite Resource Definition(XRD) and Compositions for Composite Resources created via Crossplane managed by Flux CD.
```

```yaml
- `network_landing_zone/kustomization.yaml`: Kustomization for composite resources created for Network Landing Zone.
- `network_landing_zone/nlz-namespace.yaml`: Namespace for Network Landing Zone resources.
- `network_landing_zone/nlz-xresourcegroup.yaml`: Resource Group for Network Landing Zone.
- `network_landing_zone/nlz-xsubnets.yaml`: Subnets for Network Landing Zone.
- `network_landing_zone/nlz-xvirtualnetwork.yaml`: Virtual Network for Network Landing Zone.
- `network_landing_zone/nlz-xprivatednszone_link.yaml`: Creates Private DNS Zone and links Private DNS Zone with the Vnet of Network Landing Zone.
- `network_landing_zone/nlz-xpublicip.yaml`: Creates Public IP's for Network Landing Zone.
- `network_landing_zone/nlz-xfirewall.yaml`: Creates Firewall, Firewall Route Table, Firewall Route, Firewall Policy and Firewall Policy Rule(Network & Application Firewall Rules Collection for Firewall) for Network Landing Zone.
- `network_landing_zone/nlz-xapplicationgateway.yaml`: Creates Application Gateway for Network Landing Zone.
```

```yaml
- `xrds/kustomization.yaml`: Kustomization for XRD's and composition created for Network Landing Zone resources.
- `xrds/xresourcegroups-definition.yaml`: Composite Resource Definition for Resource Group.
- `xrds/xresourcegroups-composition.yaml`: Composition for Resource Group.
- `xrds/xsubnets-definition.yaml`: Composite Resource Definition for Subnets.
- `xrds/xsubnets-composition.yaml`: Composition for Subnets.
- `xrds/xvirtualnetworks-definition.yaml`: Composite Resource Definition for Virtual Networks.
- `xrds/xvirtualnetworks-composition.yaml`: Composition for Virtual Networks.
- `xrds/xprivatednszone_link-definition.yaml`: Composite Resource Definition for Private DNS Zone and linking Private DNS Zone with Vnets.
- `xrds/xprivatednszone_link-composition.yaml`: Composition for Private DNS Zone and linking Private DNS Zone with Vnets.
- `xrds/xpublicip-definition.yaml`: Composite Resource Definition for creation of Public IP.
- `xrds/xpublicip-composition.yaml`: Composition for creation of Public IP.
- `xrds/xfirewall-definition.yaml`: Composite Resource Definition for Firewall, Firewall Route Table, Firewall Route, Firewall Policy and Firewall Policy Rule(Network & Application Firewall Rules Collection for Firewall) for Network Landing Zone.
- `xrds/xfirewall-composition.yaml`: Composition for Firewall, Firewall Route Table, Firewall Route, Firewall Policy and Firewall Policy Rule(Network & Application Firewall Rules Collection for Firewall) for Network Landing Zone.
- `xrds/xapplicationgateway-definition.yaml`: Composite Resource Definition for Application Gateway.
- `xrds/xapplicationgateway-composition.yaml`: Composition for Application Gateway.
```

### Kustomization File

crossplane/kustomization.yaml-The kustomization file includes the necessary resources for Crossplane installation:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - helm/crossplane-helmrelease.yaml
  - helm/crossplane-helmrepository.yaml
  - secrets/sealed-azure-secret.yaml
  - azure-provider/provider-config.yaml
  - azure-provider/provider-azure-network.yaml
  - azure-provider/provider-azure-management.yaml
  - resources/network_landing_zone/nlz-namespace.yaml
```

network_landing_zone/kustomization.yaml-The kustomization file includes the necessary resources used for creation of Composite Resources for Network Landing Zone via Crossplane:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - nlz-xresourcegroup.yaml
  - nlz-xvirtualnetwork.yaml
  - nlz-xsubnets.yaml
  - nlz-xpublicip.yaml
  - nlz-xfirewall.yaml
  - nlz-xprivatednszone_link.yaml
  - nlz-xapplicationgateway.yaml
```

xrds/kustomization.yaml-The kustomization file includes the necessary resources used for creation of Composite Resource Definition and Composition for Network Landing Zone via Crossplane:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - xresourcegroups-composition.yaml
  - xresourcegroups-definition.yaml
  - xvirtualnetworks-composition.yaml
  - xvirtualnetworks-definition.yaml
  - xsubnets-composition.yaml
  - xsubnets-definition.yaml
  - xpublicip-definition.yaml
  - xpublicip-composition.yaml
  - xfirewall-composition.yaml
  - xfirewall-definition.yaml
  - xprivatednszone_link-definition.yaml
  - xprivatednszone_link-composition.yaml
  - xapplicationgateway-definition.yaml
  - xapplicationgateway-composition.yaml
```

## Resources

- Sealed Secrets: [bitnami-labs/sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)
- Crossplane: [charts.crossplane.io/stable](https://charts.crossplane.io/stable)
- Compositions: [Crossplane Concepts: Compositions](https://docs.crossplane.io/latest/concepts/compositions/)
- Composite Resource Definitions: [Crossplane Concepts: Composite Resource Definitions](https://docs.crossplane.io/latest/concepts/composite-resource-definitions/)
- Composite Resources: [Crossplane Concepts: Composite Resources](https://docs.crossplane.io/latest/concepts/composite-resources/)
- Claims: [Crossplane Concepts: Claims](https://docs.crossplane.io/latest/concepts/claims/)

Feel free to explore and dive deeper into the world of Crossplane and Composite.
