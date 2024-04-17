# Bicep - parameters and variables

## Bicep parameters

### Parameters - najelepsze praktyki
- Nie twórz parametrów dla wartości, które można obliczyć na podstawie innych parametrów - lepiej unikaj niepotrzebnych parametrów, ponieważ zwiększa to złożoność, którą musimy zarządzać.
- Mają parametry, gdy wartości mogą się zmieniać między użyciami szablonu, na przykład podczas korzystania z tego samego szablonu dla różnych środowisk - np. programowania, testowania, produkcji.
- Używaj parametrów do przekazywania wpisów tajnych — nigdy nie przechowuj wpisów tajnych jako zwykłego tekstu w kodzie źródłowym, lepiej odwołuj się do wpisów tajnych z takich usług, jak Key Vault.

### Parameters - 5 typy zmiennych
- string
- object
- array
- int
- bool

### Jak deklarować parametry w pliku bicep ? - Basic
```bicep
// param myParam1 <type>
param par_my_param1 string

// param myParam2 <type> = <default_value>
param par_my_Param2 string = 'foobar'
```

### Jak deklarować parametry w pliku bicep ? - Advanced
Jako bardziej zaawansowany przypadek rozważyłbym użycie dekoratorów do wymuszania różnych ograniczeń wartości wejściowych.
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
param par_virtual_Machine_Count int = 3
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

Bicep parameters for bicep files
```bicep
using './main.bicep'

@minValue(3)
@maxValue(9)
@description('Number of virtual machines')
param par_virtual_Machine_Count int = 3

var var_storage_Prefix = 'myStorage'

param par_primary_StorageName = '${storagePrefix}Primary'
param par_secondary_StorageName = '${storagePrefix}Secondary'

```

### Constraints
- Nazwa musi być unikatowa wśród innych parametrów, zmiennych i zasobów, ale może być taka sama jak dane wyjściowe.
- W nazwie nie jest rozróżniana wielkość liter — mimo że Bicep umożliwia tworzenie dwóch parametrów, których nazwy różnią się tylko wielkością liter, zakończy się niepowodzeniem podczas wdrażania szablonu, informując, że element z tym samym kluczem został już dodany.
- Maksymalnie 256 parametrów na plik - to bardzo hojny limit, jeśli zaczniesz otrzymywać wiele parametrów, prawdopodobnie powinieneś modularyzować swój szablon.

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
W Bicep możemy również tworzyć pliki parametrów, tak jak w zwykłych starych szablonach usługi ARM, co więcej, Bicep po prostu używa plików parametrów szablonów usługi ARM i nie dodaje niczego nowego.

Tak więc pliki parametrów mogą być używane w ten sam sposób, na przykład możemy mieć main.bicep i pliki main.parameters.json.

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

New-AzResourceGroupDeployment `
  -Name myBicepTemplateDeployment `
  -ResourceGroupName rg-contoso `
  -TemplateFile ./main.bicep `
  -TemplateParameterFile ./main.parameters.bicepparams
```

```azurecli
Azure CLI
az deployment group create \
  --name myBicepTemplateDeployment \
  --resource-group rg-contoso \
  --template-file ./main.bicep \
  --parameters @main.parameters.json

Azure CLI
az deployment group create \
  --name myBicepTemplateDeployment \
  --resource-group rg-contoso \
  --template-file ./main.bicep \
  --parameters @main.parameters.bicepparams
```

## Decorators: Constraints And Metadata
W Bicep dekoratory służą do określania ograniczeń i metadanych dla parametru. Oto lista dekoratorów w Bicep:
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
param par_admin_Password string
```

#### Metadata Decorator
```bicep
@metadata({
  description: 'The number of virtual machines that will be created'
  myCustomField: 'Something important here'
})
param par_virtual_Machine_Count int
```

#### Description Decorator
```bicep
@description('The number of virtual machines that will be created')
param par_virtual_Machine_Count int
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
Jeśli moduł zwraca pewne wartości w sekcji outputs, mogą one być również używane w deklaracji zmiennej.
W poniższym przykładzie moduł zwraca pełny obiekt konta magazynu w danych wyjściowych, których następnie używamy w szablonie głównym.

```bicep
// ===== main.bicep =====

module stg './storage.bicep' = {
  name: 'storageDeployment'
  params: {
    par_storage_Account_Name: 'stcontoso'
  }
}

// Using module outputs to create a variable
var var_my_Tag = '${stg.outputs.storageAccount.kind}-${stg.outputs.storageAccount.sku.name}'

// ===== ./storage.bicep =====

param par_storage_AccountName string

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: par_storage_AccountName
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
param par_prefix string
var var_location = resourceGroup().location

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' existing = {
  name: 'stcontoso'
}

var var_Tag = '${prefix}-${location}-${stg.sku}'
```

### Use Loops
#### Creating a variable using a for-loop
```bicep
var var_secrets_Values = [for i in range(0, 3): {
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
- Nazwa musi być unikatowa wśród parametrów, zasobów, modułów i innych zmiennych, ale może być taka sama jak nazwa wyjściowa
- Nie można ponownie przypisać innej wartości do zmiennej (przypisać tylko raz)
- Nie można rozdzielić deklaracji i inicjalizacji