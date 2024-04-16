# Bicep - parameters and variables

## Bicep parameters

### Parameters - Best practices
- Don’t create parameters for values that can be computed based on other parameters - better avoid unnecessary parameters since it adds complexity we need to manage.
- Do have parameters when values can change between template usages, for example, when using the same template for different environments - e.g. development, test, production.
- Do use parameters to pass secrets - never store secrets as plain text in source code, better reference secrets from such services as Key Vault.

### Parameters - 5 types available
- string
- object
- array
- int
- bool

### How to use declare parameters? - Basic
```bicep
// param myParam1 <type>
param myParam1 string

// param myParam2 <type> = <default_value>
param myParam2 string = 'foobar'
```

### How to use declare parameters? - Advanced
As a more advanced case I’d consider the use of decorators to enforce different constraints on input values.
```bicep
// @decorator1()
// @decorator2()
// ...
// param myParam <type> = <default_value>

@minValue(3)
@maxValue(9)
@description('Number of virtual machines')
// OR:
// @metadata({
//   description: 'Number of virtual machines'
// })
param virtualMachineCount int = 3
```

Bicep parameters to ARM Template
```json
{
  "...": "...",
  "parameters": {
    "virtualMachineCount": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Number of virtual machines"
      },
      "maxValue": 9,
      "minValue": 3
    }
  },
  "...": "..."
}
```

### Constraints
- Name must be unique among other parameters, variables, and resources, but can be the same as outputs.
- Name is case-insensitive - even though Bicep allows creating two parameters whose names only differ in case, it will fail during template deployment telling that item with the same key has already been added.
- Maximum 256 parameters per file - it’s a very generous limit, if you start getting many parameters then you should probably modularize your template.

### Where To Declare Parameters?
In Bicep we can declare parameters in any place, it doesn’t actually matter.

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  ...
}

param storageAccountName string
```

### Sample Bicep Storage Account
```bicep
param storageAccountName string = 'stcontoso'
param storageAccountSettings object = {
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: 'Standard_LRS'
}

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  location: storageAccountSettings.location
  kind: storageAccountSettings.kind
  sku: {
    name: storageAccountSettings.sku
  }
}
```

### Sample - literal value
```bicep
param myString string = 'foobar'
param myInt int = 28
param myBool bool = true
param myArr array = [
  'one'
  'two'
  'three'
]
param myObj object = {
  name: 'John'
  age: 42
}
```

### Sample - Another Parameter
```bicep
param namePrefix string = ''
param storageAccountName string = '${namePrefix}staccount1'
param vmssName string = '${namePrefix}vmss1'
```

### Sample - Function/Expression
```bicep
param location string = resourceGroup().location
```

### Passing Parameters
#### Azure PowerShell
```powershell
New-AzResourceGroupDeployment `
  -Name myBicepTemplateDeployment `
  -ResourceGroupName rg-contoso `
  -TemplateFile ./main.bicep `
  -storageAccountName stcontosobackup
```

#### Azure CLI
```azurecli
az deployment group create \
  --name myBicepTemplateDeployment \
  --resource-group rg-contoso \
  --template-file ./main.bicep \
  --parameters storageAccountName=stcontosobackup
```

## Parameter File
In Bicep we also can create parameter files like in plain old ARM templates, moreover, Bicep just uses ARM template parameter files and doesn’t add anything new.

So, parameter files can be used in the same way, for example, we could have main.bicep and main.parameters.json files.

main.bicep
```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string
```

main.parameters.json
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "value": "stcontoso"
    }
  }
}
```

```powershell
Powershell
New-AzResourceGroupDeployment `
  -Name myBicepTemplateDeployment `
  -ResourceGroupName rg-contoso `
  -TemplateFile ./main.bicep `
  -TemplateParameterFile ./main.parameters.json
```

```azurecli
Azure CLI
az deployment group create \
  --name myBicepTemplateDeployment \
  --resource-group rg-contoso \
  --template-file ./main.bicep \
  --parameters @main.parameters.json
```

## Decorators: Constraints And Metadata
In Bicep decorators are used to specify constraints and metadata for a parameter. Here’s a list of decorators in Bicep:
- allowed
- secure
- minLength & maxLength
- minValue & maxValue
- description
- metadata

### Examples
#### Syntax
```bicep
@secure()
@minLength(8)
@maxLength(24)
param adminPassword string
```

#### Metadata Decorator
```bicep
@metadata({
  description: 'The number of virtual machines that will be created'
  myCustomField: 'Something important here'
})
param virtualMachineCount int
```

#### Description Decorator
```bicep
@description('The number of virtual machines that will be created')
param virtualMachineCount int
```

Description decorator is a less verbose way to specify metadata object with description. If we only want to specify description, I’d rather use description decorator instead of metadata.

### String Parameters
String interpolation is a convenient and succinct way of combining literals with dynamic values.

In a single line string an expression should be put between curly braces of ${} like this: '${expression}'.

In ARM templates we had to use format or concat functions to achieve similar results, but string interpolation results in more readable code.

```bicep
param resourceTag string = '${resourceGroup().name}-${resourceGroup().location}-resource'
```

### Multi line strings
In Bicep we can specify strings that span multiple lines by specifying our multiline string between three single quotes '''string is here'''.

```bicep
param myParam string = '''
First line
Second line
Third line'''
```

### Providing Allowed Values
```bicep
@allowed([
  'Standard_ZRS'
  'Standard_GRS'
  'Standard_RAGRS'
])
param skuName string
```

### Minimum & Maximum Value
```bicep
@minValue(3)
@maxValue(9)
param virtualMachineCount int = 5
```

### Specifying Allowed Values
```bicep
@allowed([
  3
  5
  7
  9
])
param virtualMachineCount int
```

### Boolean Parameters
```bicep
param virtualMachineCount int
param isLargeScaleSet bool = virtualMachineCount > 100
```

### Array Parameters
```bicep
@minLength(1)
@maxLength(5)
param storageAccountNames array
```

### Object Parameters
Objects have similar limitation to arrays in regards to declaration on multiple lines since Bicep uses newlines as a separator.

Below is an example of an object parameter with a default value.
```bicep
param storageAccountSettings object = {
  location: 'West US'
  sku: 'Standard_GRS'
  kind: 'StorageV2'
}
```

## Bicep Variables
### Variables - when to use
- Reuse the same value in multiple places in the file
- Be able to easily change value used in multiple places
- Declare a complex expression in a variable to improve readability

### Syntax
- No need to specify type - it is inferred automatically from the assigned value
- On the right side of the assignment can be any expression
- Value can be null as well

```bicep
// var <name> = <expression>
var myString = 'some string value'
var myNull = null
var location = resourceGroup().location
```

### Declare Anywhere In File
```bicep
param storageAccountName string

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  kind: 'StorageV2'
  location: resourceLocation
  sku: {
    name: 'Standard_LRS'
  }
}
```

// Even though it is possible to declare variable anywhere
// Still worth declaring it before the place it is used for better readability
```bicep
var resourceLocation = resourceGroup().location
```

The compiled ARM template will have our variable resourceLocation in the variables section:
```json
{
  "...": "...",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    }
  },
  "variables": {
    "resourceLocation": "[resourceGroup().location]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[parameters('storageAccountName')]",
      "kind": "StorageV2",
      "location": "[variables('resourceLocation')]",
      "sku": {
        "name": "Standard_LRS"
      }
    }
  ]
}
```
### Use Parameters, Resources, Modules, or Another Variables
#### Parameters
```bicep
param name string
var greeting = 'Hello, ${name}!'  // string interpolation
```

#### Variables
```bicep
var name = 'John'
var greeting = 'Hello, ${name}!'
```

#### Resources
```bicep
resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' existing = {
  name: 'stcontoso'
}

var myTag = '${stg.type}-${stg.kind}-${stg.sku.name}'
```

#### Modules
If a module returns some values in outputs section, they can also be used in variable declaration.
In the example below module returns full storage account object in outputs which we then use in the main template.

```bicep
// ===== main.bicep =====

module stg './storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'stcontoso'
  }
}

// Using module outputs to create a variable
var myTag = '${stg.outputs.storageAccount.kind}-${stg.outputs.storageAccount.sku.name}'

// ===== ./storage.bicep =====

param storageAccountName string

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

output storageAccount object = stg
```
### Use Functions
```bicep
param prefix string
var location = resourceGroup().location

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' existing = {
  name: 'stcontoso'
}

var myTag = '${prefix}-${location}-${stg.sku}'
```

### Use Loops
#### Creating a variable using a for-loop
```bicep
var secretsValues = [for i in range(0, 3): {
  name: 'secret${i}'
  value: 'supersecretvalue${i}'
}]
```

#### Assuming that a key vault already exists
```bicep
resource kv 'Microsoft.KeyVault/vaults@2019-09-01' existing = {
  name: 'kv-contoso'
}
```

#### Using variable to create multiple resources
```bicep
resource secrets 'Microsoft.KeyVault/vaults/secrets@2019-09-01' = [for secret in secretsValues: {
  name: secret.name
  parent: kv
  properties: {
    value: secret.value
  }
}]
```

### Constraints & Limitations
- Name must be unique among parameters, resources, modules, and other variables, but it can be the same as output name
- Cannot reassign another value to a variable (assign only once)
- Declaration and initialization can’t be separated
