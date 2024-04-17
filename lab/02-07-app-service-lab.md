## Create an Azure App Service with Bicep

### Objective
Your task is to create an Azure App Service using Bicep, an Infrastructure as Code (IaC) language for deploying Azure resources. This App Service should have a specific application setting (`TEST1` with the value `value`) and be configured to only accept traffic from your IP address using `ipSecurityRestrictions`.

### Requirements

1. **Azure App Service Plan**: Create an Azure App Service Plan. The specifics of the pricing tier and region are up to you, but it should be compatible with the App Service you intend to deploy.
2. **Azure App Service**: Deploy an Azure App Service within the App Service Plan you created.
3. **AppSettings**: Configure the App Service to include an application setting named `TEST1` with the value `value`.
4. **IP Security Restrictions**: Restrict access to the App Service so that only requests from your specific IP address are allowed.

### Instructions

1. Install the necessary tools if you haven't already: Azure CLI and Bicep.
2. Create a new Bicep file named `appservice.bicep`.
3. Define the resources and properties required to meet the above requirements in your Bicep file.
4. Deploy your Bicep file using the Azure CLI.

### Helpful Resources

- [Bicep Documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)
- [Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/)

<details>
<summary>Solution</summary>

Here's a basic example of how you might define your `appservice.bicep` file to meet the requirements:

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

To deploy this Bicep file:

1. Open your terminal or command prompt.
2. Navigate to the directory containing your `appservice.bicep` file.
3. Run the following command, replacing `<resource-group-name>` with the name of your Azure resource group:

```shell
az login
az deployment group create `
    --resource-group rg-test-app-msdasdas `
    --template-file main.bicep `
    --parameters main.bicepparam `
    --confirm-with-what-if
```

This command will deploy the resources defined in your Bicep file to your Azure subscription.

</details>