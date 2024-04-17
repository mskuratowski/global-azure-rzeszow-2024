## Utwórz Storage Account za pomocą Bicep

### Cel
Twoim zadaniem jest utworzenie usługi Azure Storage Account przy użyciu Bicep.

### Wymagania

Storage Account powinien mieć następujące właściwości:

- Nazwa: `Dowolna`
- Typ konta: StorageV2 (General Purpose v2)
- Replication: LRS (Locally-Redundant Storage)
- Dostęp publiczny: Włączony
- TLS: 1.2

### Instrukcje

1. Zainstaluj niezbędne narzędzia, jeśli jeszcze tego nie zrobiłeś: Azure CLI i Bicep.
2. Utwórz nowy plik Bicep o nazwie `storage.bicep`.
3. Zdefiniuj zasoby i właściwości wymagane do spełnienia powyższych wymagań w pliku Bicep.
4. Wykonaj wdrożenie.

### Pomocne zasoby

- [Bicep Dokumentacja](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure CLI Dokumentacja](https://docs.microsoft.com/en-us/cli/azure/)

<details>
<summary>Rozwiązanie</summary>

Oto podstawowy przykład tego, jak można zdefiniować plik `storage.bicep`, aby spełnić wymagania:

```bicep
// --------------------------------------------------------------------------------
// PARAMETERS
// --------------------------------------------------------------------------------
param storageAccountName string = uniqueString('mskuratowski')
param location string = 'westeurope'

// --------------------------------------------------------------------------------
// RESOURCES
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    allowBlobPublicAccess: true
    minimumTlsVersion: 'TLS1_2'
  }
}
```

Aby wdrożyć:

1. Otwórz terminal.
2. Przejdź do katalogu zawierającego plik `storage.bicep`.
3. Uruchom następujące polecenie, zastępując `<resource-group-name>` nazwą grupy zasobów Azure:

```shell
az login
az deployment group create `
    --resource-group <<RG_NAME>> `
    --template-file main.bicep `
    --parameters main.bicepparam `
    --confirm-with-what-if
```

To polecenie wdroży zasoby zdefiniowane w pliku Bicep w subskrypcji Azure.


</details>