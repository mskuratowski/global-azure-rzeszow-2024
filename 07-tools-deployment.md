# Narzędzia do zarzazania w Azure
## Przydatne linki

- [Bicep Documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/)
- [Azure PowerShell module](https://learn.microsoft.com/en-us/powershell/module/az.accounts/?view=azps-11.5.0)

## Portal Azure

## Azure CLI


```bash

az login # logwanie do portalu Azure

az deployment group create # wdrożenie na poziomie grupy zasobów 
az deployment mg create # wdrożenie na poziomie grupy zarządzalnej 
az deployment sub create # wdrożenie na poziomie subskrypcji
az deployment tenant create # # wdrożenie na poziomie konta / Entra ID
```
Demo

## Azure PowerShell

```powershell

Connect-AzAccount # logwanie do portalu Azure

New-AzResourceGroupDeployment # wdrożenie na poziomie grupy zasobów 
New-AzManagementGroupDeployment # wdrożenie na poziomie grupy zarządzalnej 
New-AzSubscriptionDeployment # wdrożenie na poziomie subskrypcji
New-AzTenantDeployment # # wdrożenie na poziomie konta / Entra ID

```

Demo

## Azure Cloud Shell

Demo