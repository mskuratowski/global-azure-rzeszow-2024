## Utwórz Azure App Service za pomocą Bicep

### Cel
Twoim zadaniem jest utworzenie usługi Azure App Service przy użyciu Bicep. Azure App Service powinna mieć określone 'application setting' (`TEST1` z wartością `value`) i być skonfigurowana tak, aby akceptować ruch tylko z twojego adresu IP przy użyciu `ipSecurityRestrictions`.

### Wymagania

1. **Azure App Service Plan**: Utwórz plan usługi Azure App Service. Specyfika warstwy cenowej i regionu zależy od Ciebie, ale powinna być zgodna z usługą App Service, którą zamierzasz wdrożyć.
2. **Azure App Service**: Wdrożenie usługi Azure App Service w ramach utworzonego planu App Service.
3. **AppSettings**: Skonfiguruj usługę App Service tak, aby zawierała ustawienie aplikacji o nazwie `TEST1` z wartością `value`.
4. **IP Security Restrictions**: Ograniczenie dostępu do App Service tak, aby dozwolone były tylko żądania z określonego adresu IP użytkownika.

### Instrukcje

1. Zainstaluj niezbędne narzędzia, jeśli jeszcze tego nie zrobiłeś: Azure CLI i Bicep.
2. Utwórz nowy plik Bicep o nazwie `appservice.bicep`.
3. Zdefiniuj zasoby i właściwości wymagane do spełnienia powyższych wymagań w pliku Bicep.
4. Wykonaj wdrożenie.

### Pomocne zasoby

- [Bicep Dokumentacja](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure App Service Dokumentacja](https://docs.microsoft.com/en-us/azure/app-service/)
- [Azure CLI Dokumentacja](https://docs.microsoft.com/en-us/cli/azure/)

<details>
<summary>Rozwiązanie</summary>

Oto podstawowy przykład tego, jak można zdefiniować plik `appservice.bicep`, aby spełnić wymagania:

```bicep
// --------------------------------------------------------------------------------
// PARAMETERS
// --------------------------------------------------------------------------------
@description('Location for all resources.')
param location string = resourceGroup().location

@description('Azure App Service App Name')
param appName string  = 'app-global-azure-workshops-12345'

@description('The SKU of App Service Plan.')
param sku string = 'F1'

@description('The Runtime stack of current web app')
param linuxFxVersion string = 'DOTNETCORE|8.0'

// --------------------------------------------------------------------------------
// VARIABLES
// --------------------------------------------------------------------------------
var planName = 'plan-${appName}'

// --------------------------------------------------------------------------------
// RESOURCES
// --------------------------------------------------------------------------------
resource plan 'Microsoft.Web/serverfarms@2023-01-01' = {
  name: planName
  location: location
  sku: {
    name: sku
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

resource app 'Microsoft.Web/sites@2023-01-01' = {
  name: appName
  location: location
  properties: {
    httpsOnly: true
    serverFarmId: plan.id
    siteConfig: {
      linuxFxVersion: linuxFxVersion
      minTlsVersion: '1.2'
      ftpsState: 'FtpsOnly'
      appSettings: [
        {
          name: 'TEST1'
          value: 'abc'
        }
      ]
      ipSecurityRestrictions: [
        {
          ipAddress: '91.246.67.157/32'
          action: 'Allow'
        }
      ]
    }
  }
  identity: {
    type: 'SystemAssigned'
  }
}

// --------------------------------------------------------------------------------
// OUTPUTS
// --------------------------------------------------------------------------------
output appServiceUrl string = 'https://${app.properties.defaultHostName}'
```

Aby wdrożyć:

1. Otwórz terminal.
2. Przejdź do katalogu zawierającego plik `appservice.bicep`.
3. Uruchom następujące polecenie, zastępując `<resource-group-name>` nazwą grupy zasobów Azure:

```shell
az login # logowanie

az group create --location "WestEurope" --name "rg-test-app-msdasdas" ## stworzenie resource group

az deployment group create `
    --resource-group rg-test-app-msdasdas `
    --template-file main.bicep `
    --parameters main.bicepparam `
    --confirm-with-what-if
```

To polecenie wdroży zasoby zdefiniowane w pliku Bicep w subskrypcji Azure.


</details>