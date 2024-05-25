# Helm Templates for Crossplane XRCs

This README provides instructions for developing configurable Helm templates to abstract and deploy kishore Test infra azure resources provisioned via Crossplane. The aim is to simplify the management and deployment of Crossplane resources in Kubernetes using Helm, allowing for easy adaptation to future infrastructure changes.

## References

- [Helm_Chart_Template_Guide](https://helm.sh/docs/chart_template_guide/getting_started/)

## Overview

The configuration steps involve:

1. **Define Helm Chart Structure**: Create a new Helm chart in the project repository, adhering to the recommended directory structure.
2. **Develop Helm Templates:**: Develope Helm templates for different types of resources provisioned via Crossplane for Network Landing Zone(kishore).
3. **Parameterize Helm Templates**: Parameterize the Helm templates using Go Templating format.


## Configuration Steps

### 1. Define Helm Chart Structure

Creating a new Helm chart involves setting up a directory structure and configuration files according to Helm's recommended practices as below,

```yaml
xrc/
    - `Chart.yaml`: Contains a description of the chart.
    - `values.yaml`: Default values for a chart.
    - `values.schema.json`: JSON Schema, which defines the structure and required values.
    - `templates/`: Directory for template files.
```

### 2. Develop Helm Templates

Developing Helm templates involves creating YAML files with Go templating directives to define the Kubernetes resources that will be managed by the Helm chart.These templates serve as the blueprint for the actual Kubernetes resources that will be deployed to the cluster like resource group, virtual network, subnet, public ip, firewall, private dns zone, application gateway etc.

Within the templates directory of Helm chart, create individual YAML files for each resource type you need to provision, as below,

```yaml
templates/
    - `xresourcegroup-claim.yaml`
    - `xvirtualnetwork-claim.yaml`
    - `xsubnets-claim.yaml`
    - `xpublicip-claim.yaml`
    - `xfirewall-claim.yaml`
    - `xprivatednszone_link-claim.yaml`
    - `xapplicationgateway-claim.yaml`
```

### 3. Parameterize Helm Templates

Using Go templating to parameterize the configurations of resources. This allows users to customize the resource settings by providing values through the Helm chart's `values.yaml`.

General syntax of Go templating considering Resource Group as example:

`xresourcegroup-claim.yaml`

```yaml
apiVersion: azureinfra.dockerdesktop.kishorekumar.today/v1alpha1
kind: xResourceGroupClaim
metadata:
  name: {{ .Values.resourceGroup.name }}-claim
  namespace: {{ .Values.namespace }}
spec:
  compositionRef:
    name: xresourcegroups-azure
  parameters:
    location: {{ .Values.resourceGroup.location }}
    providerConfigRef: 
      name: {{ .Values.providerConfigRef }} 
```

-Actions: <br>
    `{{ }}`: These double curly braces denote an action block where Go templating expressions are evaluated.

-Variables: <br>
    `.Values`: This refers to the values passed into the Helm chart. Helm allows us to define values in a values.yaml file or override them at runtime using --set flags.<br>
    `.Values.resourceGroup.name`: This accesses the name property under the resourceGroup key in the Helm values(values.yaml file).

-Pipelines: Pipelines are sequences of commands separated by the pipe character |. They pass the result of one command as input to the next command.

-Control Structures: Go templates support control structures like conditionals (if, else, range), allowing us to execute logic based on conditions or iterate over collections.

-Functions: Functions can be called within templates to perform specific actions or manipulate data. Some built-in functions are provided by default, and we can also define custom functions.

-Comments: Comments in Go templates start with {{/* and end with */}}.