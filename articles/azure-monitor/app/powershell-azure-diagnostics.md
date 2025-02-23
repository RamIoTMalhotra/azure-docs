---
title: Using PowerShell to setup Application Insights in an Azure  | Microsoft Docs
description: Automate configuring Azure Diagnostics to pipe data to Application Insights.
ms.topic: conceptual
ms.date: 08/06/2019 
ms.custom: devx-track-azurepowershell
---

# Using PowerShell to set up Application Insights for Azure Cloud Services

[Microsoft Azure](https://azure.com) can be [configured to send Azure Diagnostics](../agents/diagnostics-extension-to-application-insights.md) to [Azure Application Insights](./app-insights-overview.md). The diagnostics relate to Azure Cloud Services and Azure VMs. They complement the telemetry that you send from within the app using the Application Insights SDK. As part of automating the process of creating new resources in Azure, you can configure diagnostics using PowerShell.

## Azure template
If the web app is in Azure and you create your resources using an Azure Resource Manager template, you can configure Application Insights by adding this to the resources node:

```json
{
  resources: [
    /* Create Application Insights resource */
    {
      "apiVersion": "2015-05-01",
      "type": "microsoft.insights/components",
      "name": "nameOfAIAppResource",
      "location": "centralus",
      "kind": "web",
      "properties": { "ApplicationId": "nameOfAIAppResource" },
      "dependsOn": [
        "[concat('Microsoft.Web/sites/', myWebAppName)]"
      ]
    }
  ]
}
``` 

* `nameOfAIAppResource` - a name for the Application Insights resource
* `myWebAppName` - the ID of the web app

## Enable diagnostics extension as part of deploying a Cloud Service
The `New-AzureDeployment` cmdlet has a parameter `ExtensionConfiguration`, which takes an array of diagnostics configurations. These can be created using the `New-AzureServiceDiagnosticsExtensionConfig` cmdlet. For example:

```azurepowershell
$service_package = "CloudService.cspkg"
$service_config = "ServiceConfiguration.Cloud.cscfg"
$diagnostics_storagename = "myservicediagnostics"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$primary_storagekey = (Get-AzStorageKey `
  -StorageAccountName "$diagnostics_storagename").Primary
$storage_context = New-AzStorageContext `
  -StorageAccountName $diagnostics_storagename `
  -StorageAccountKey $primary_storagekey

$webrole_diagconfig = `
  New-AzureServiceDiagnosticsExtensionConfig `
    -Role "WebRole" -Storage_context $storageContext `
    -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = `
  New-AzureServiceDiagnosticsExtensionConfig `
    -Role "WorkerRole" `
    -StorageContext $storage_context `
    -DiagnosticsConfigurationPath $workerrole_diagconfigpath

  New-AzureDeployment `
    -ServiceName $service_name `
    -Slot Production `
    -Package $service_package `
    -Configuration $service_config `
    -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)
``` 

## Enable diagnostics extension on an existing Cloud Service
On an existing service, use `Set-AzureServiceDiagnosticsExtension`.

```azurepowershell
$service_name = "MyService"
$diagnostics_storagename = "myservicediagnostics"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"
$primary_storagekey = (Get-AzStorageKey `
  -StorageAccountName "$diagnostics_storagename").Primary
$storage_context = New-AzStorageContext `
  -StorageAccountName $diagnostics_storagename `
  -StorageAccountKey $primary_storagekey

Set-AzureServiceDiagnosticsExtension `
  -StorageContext $storage_context `
  -DiagnosticsConfigurationPath $webrole_diagconfigpath `
  -ServiceName $service_name `
  -Slot Production `
  -Role "WebRole" 
Set-AzureServiceDiagnosticsExtension `
  -StorageContext $storage_context `
  -DiagnosticsConfigurationPath $workerrole_diagconfigpath `
  -ServiceName $service_name `
  -Slot Production `
  -Role "WorkerRole"
```

## Get current diagnostics extension configuration

```azurepowershell
Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```


## Remove diagnostics extension

```azurepowershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

If you enabled the diagnostics extension using either `Set-AzureServiceDiagnosticsExtension` or `New-AzureServiceDiagnosticsExtensionConfig` without the Role parameter, then you can remove the extension using `Remove-AzureServiceDiagnosticsExtension` without the Role parameter. If the Role parameter was used when enabling the extension then it must also be used when removing the extension.

To remove the diagnostics extension from each individual role:

```azurepowershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```


## See also
* [Monitor Azure Cloud Services apps with Application Insights](./cloudservices.md)
* [Send Azure Diagnostics to Application Insights](../agents/diagnostics-extension-to-application-insights.md)


