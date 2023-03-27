# ARM-templates

Important note:  Keys in the template JSON are recognized by the Azure Reource Manager(ARM), so the keys have to match exactly what ARM is looking for. 

## Create a resource group with ARM-template
Start with a deployment with not resources

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


## Add a resource to ARM-template  (The "resource" key in template)
Every resource you deploy has at least the following three properties:

- type: Type of the resource. This value is a combination of the namespace of the resource provider and the resource type such as Microsoft.Storage/storageAccounts.
- apiVersion: Version of the REST API to use for creating the resource. Each resource - - provider publishes its own API versions, so this value is specific to the type.
name: Name of the resource.

Two important porints here:
- Many of the properties are optional. No need to add all of them.
- The resource provider could release a new API version, but the version you're deploying doesn't have to change. 

```
# find parameters for each resource here
https://learn.microsoft.com/en-us/azure/templates/

# for example the required parameters for storage account can be found there
https://learn.microsoft.com/en-us/azure/templates/microsoft.storage/storageaccounts?pivots=deployment-language-arm-template

# Add the following resource in json

    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-09-01",
      "name": "myteststorageaccount0326",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    }

# Deploy template
templateFile="templates/azuredeploy.json"
az deployment group create \
  --name addstorage \
  --resource-group mytestResourceGroup \
  --template-file $templateFile



# Verify deployment

Option1: Azure portal > Resource groups > select myResourceGroup > setect mytestResourceGroup > in the Essentials section, search for "Deployments: 2 Succeeded".

Option2: zure portal > storage sccount> check if "myteststorageaccount0326" is over there in the list of storage accounts

```

## Make templates reusable: Add parameters to ARM template (The "parameter" key in template)
Note here the brackets means a template expression, where `parameters` is a function.

```
Add parameter section to template

  "parameters": {
    "storageName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    }
  },


change value for name 

  "name": "[parameters('storageName')]",
```

If the resource already exists, and there's no change in the properties, there's no need for further action. If the resource exists and a property changes, it updates. If the resource doesn't exist, it's created.

```
az deployment group create \
  --name addnameparameter \
  --resource-group mytestResourceGroup \
  --template-file $templateFile \
  --parameters storageName=myteststorageaccount0326
```

Redeploy with sku as input argument for template
```
az deployment group create \
  --name usenondefaultsku \
  --resource-group mytestResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=Standard_GRS storageName=myteststorageaccount0326
```

## Make templates reusable: Further have location function in parameters

Template json

```
Add location to parameters
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }

and change location value to 
    "location": "[parameters('location')]",
```

Deploy

```
az deployment group create \
  --name usenondefaultsku \
  --resource-group mytestResourceGroup \
  --template-file $templateFile \
  --parameters storageSKU=Standard_GRS storageName=myteststorageaccount0326
```

## Make templates reusable: Having variables  (The "variables" key in template)
Add variables (not sth you wanna set manually but will be changed based on the parameters and other values in the template)
and remove the storage name in parameters, we do this because we dont want to manually set the unique storage name
Again concat() and uniqueString() are two functions.

```
Change storage name to storage prefix in parameters

    "storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    },

Add variable section

  "variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
  },

change resources.name

"name": "[variables('uniqueStorageName')]",
```


Deploy

```
az deployment group create \
  --name addnamevariable \
  --resource-group mytestResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

## Make templates return outputs (the "outputs" key in template)
Use the `reference()` function to get return values from the template. One can set an "outputs" key in the json with values with reference function.

```
Add outputs section in json

  "outputs": {
    "storageEndpoint": {
      "type": "object",
      "value": "[reference(variables('uniqueStorageName')).primaryEndpoints]"
    }
  }

```


Deploy
```
az deployment group create \
  --name addoutputs \
  --resource-group mytestResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

Note that the outputs will show up in the UI.