# Bicep - Files structure and namings

## Struktura pliku *.bicep

Dla wszystkich plików Bicep utworzonych w ramach tego projektu będą one zgodne ze wzorcem struktury grupowania według typu elementu, co pokazano na poniższym przykładzie.

```bicep
/*
SUMMARY: An example deployment of a resource group.
DESCRIPTION: Deploy a resource group to UK south taking a naming prefix as it's only parameter.
AUTHOR/S: Michal Machniak
VERSION: 1.0.0
*/


// SCOPE
targetScope = 'subscription' //Deploying at Subscription scope to allow resource groups to be created and resources in one deployment


// PARAMETERS
@description('Example description for parameter. - DEFAULT VALUE: "TEST"')
param par_Example_Resource_GroupName_Prefix string = 'TEST'


// VARIABLES
var var_Example_Resource_GroupName = 'rsg-${par_Example_Resource_GroupName_Prefix}' // Create name for the example resource group


// RESOURCE DEPLOYMENTS 
resource res_Example_ResourceGroup 'Microsoft.Resources/resourceGroups@2021-04-01' = {
  name: var_Example_Resource_GroupName
  location: 'uksouth'
}




// OUTPUTS
output Resource_GroupExampleID string = res_Example_ResourceGroup.id

```


## Konwencja nazewnicza

Przykład struktury nazewniczej w celu utrzymania porządku w kodzie oraz referencjach.

| Element Type | Naming Prefix | Example                                                              |
| :----------: | :-----------: | :------------------------------------------------------------------- |
|  Parameters  |     `par`     | `par_Location`, `par_Management_GroupsName_Prefix`                       |
|  Variables   |     `var`     | `var_Condition_Expression`, `var_Intermediate_Root_Management_GroupName`   |
|  Resources   |     `res`     | `res_Intermediate_Root_ManagementGroup`, `res_ResourceGroup_LogAnalytics` |
|   Modules    |     `mod`     | `mod_ManagementGroups`, `mod_LogAnalytics`                             |
|   Outputs    |     `N/A`     | `Intermediate_Root_ManagementGroupID`, `LogAnalytics_WorkspaceID` |

## Struktura plików i folderów 

Układ plików fizycznych i katalogów dla kodu infrastruktury, układ ten nie ma wpływu na proces wgrywania jest to tylko kwestia porządkowa

| Nazwa Katalogu | Poziom Folderu |
|----|----|
|.modules|0|
|.pipelines|0|
|.parameters|0|
|.scripts|0|


<p align="center">
  <img src="./assest/az-10.png" />
</p>


## Bicep config - co  to jest

Bicep obsługuje opcjonalny plik konfiguracji o nazwie bicepconfig.json. W tym pliku możesz dodać wartości, które dostosują środowisko deweloperskie Bicep. Ten plik jest scalony z domyślnym plikiem konfiguracji. Aby uzyskać więcej informacji, zobacz Omówienie procesu scalania. Aby dostosować konfigurację, utwórz plik konfiguracji w tym samym katalogu lub katalogu nadrzędnym plików Bicep. Jeśli istnieje wiele katalogów nadrzędnych zawierających bicepconfig.json pliki, Bicep używa konfiguracji z najbliższej. Aby uzyskać więcej informacji, zobacz Omówienie procesu rozpoznawania plików.

```json
            {
              "analyzers": {
                "core": {
                  "enabled": true,
                  "verbose": true,
                  "rules": {
                    "adminusername-should-not-be-literal": {
                      "level": "error"
                    },
                    "no-hardcoded-env-urls": {
                      "level": "error"
                    },
                    "no-unnecessary-dependson": {
                      "level": "error"
                    },
                    "no-unused-params": {
                      "level": "error"
                    },
                    "no-unused-vars": {
                      "level": "error"
                    },
                    "outputs-should-not-contain-secrets": {
                      "level": "error"
                    },
                    "prefer-interpolation": {
                      "level": "error"
                    },
                    "secure-parameter-default": {
                      "level": "error"
                    },
                    "simplify-interpolation": {
                      "level": "error"
                    },
                    "protect-commandtoexecute-secrets": {
                      "level": "error"
                    },
                    "use-stable-vm-image": {
                      "level": "error"
                    },
                    "explicit-values-for-loc-params": {
                      "level": "error"
                    },
                    "no-hardcoded-location": {
                      "level": "error"
                    },
                    "no-loc-expr-outside-params": {
                      "level": "error"
                    },
                    "max-outputs": {
                      "level": "error"
                    },
                    "max-params": {
                      "level": "error"
                    },
                    "max-resources": {
                      "level": "error"
                    },
                    "max-variables": {
                      "level": "error"
                    }
                  }
                }
              }
            }
      ```


## Co  to jest moduł
Założenie:
- Moduły powinny być nie zależene
- Moduły zwierają powtarzalne zasoby i połączone zasoby
- Nie należy zagnieżdzać modułów

### Moduł dla resource group

```bicep

targetScope = 'subscription'
param par_location string = deployment().location
param par_rg_name string
param par_tags object
resource res_rg 'Microsoft.Resources/resourceGroups@2020-06-01' = {
  location: par_location
  name: par_rg_name
  tags: par_tags
}
output out_rg_id string = empty(res_rg) == false ? res_rg.id : ''
output out_rg_name string = empty(res_rg) == false ? res_rg.name : ''
output out_rg_tags object = empty(res_rg) == false ? res_rg.tags : {}



```

### Przykład użycia modułu

```bicep
/*
SUMMARY: An example deployment of a resource group.
DESCRIPTION: Deploy a resource group to UK south taking a naming prefix as it's only parameter.
AUTHOR/S: Michal Machniak
VERSION: 1.0.0
*/


// SCOPE
targetScope = 'subscription' //Deploying at Subscription scope to allow resource groups to be created and resources in one deployment


// PARAMETERS
@description('Example description for parameter. - DEFAULT VALUE: "TEST"')
param par_Example_Resource_GroupName_Prefix string = 'TEST'


// VARIABLES
var var_Example_Resource_GroupName = 'rsg-${par_Example_Resource_GroupName_Prefix}' // Create name for the example resource group


// RESOURCE DEPLOYMENTS WITH MODULE
module mod_rg_network '.modules/resource-group/rg.bicep' = {
    name: var_Example_Resource_GroupName
    params: {
        par_tags: par_tags_default
        par_rg_name: var_rg_name_network
    }
}


// OUTPUTS
output Resource_GroupExampleID string = res_Example_ResourceGroup.id

```

### Struktura folderów dla modułów


<p align="center">
  <img src="./assest/az-11.png" />
</p>
