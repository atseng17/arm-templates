# ARM-templates

## Example of creating a resource group with ARM-template

```
# log on to azure portal
az login

# choose the subscription to use
az account set --subscription 21164e0c-426b-4e6f-b4e2-b45a9b1638d1

# Create resource group

az group create \
  --name mytestResourceGroup \
  --location "East US"


# Deploy template

templateFile="templates/azuredeploy.json"
az deployment group create \
  --name blanktemplate \
  --resource-group mytestResourceGroup \
  --template-file $templateFile

# Verify deployment

Option1: check output from "az deployment group create ..." and search for "provisioningState": "Succeeded"

Option2: Azure portal > Resource groups > select myResourceGroup > setect mytestResourceGroup > in the Essentials section, search for "Deployments: 1 Succeeded".

# Clean up resources

Azure portal > Resource groups > Check the box next to myResourceGroup and select mytestResourceGroup > Delete 
```