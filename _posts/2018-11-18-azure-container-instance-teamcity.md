---
title: Create and remove Azure Container Instances from a TeamCity build
author: Matthew Davis
date: 2018-11-18
toc: true
excerpt: 
categories:
    - powershell
tags:
    - powershell
    - azure
    - teamcity
    - aci
    - azure container instances
published: true
---

# Introduction

Create a container in Azure Container Instance Groups from a TeamCity build job for testing.

The requirement came up recently to be able to test certain steps of a build against an application running in a linux container. The build agents used run on EC2 and couldn't do nested virtualisation so the original plan of using docker on the agent to run a linux container to test against didn't work due to running linux containers on Windows requires [hyper-v to be installed].

I looked around at some other solutions and watched the [channel9 video on Azure container instances] and thought they could fit the need.

After checking the [Azure Container Instances docs] then the how to get started with [PowerShell section] and giving it a go, it is simple to get a container up and running and remove it:

```powershell
# Create Nginx Container with date time for unique DNS name

$date = Get-Date -Format yyyyMMddHHMMss
$ContainerGroupName = "demoContainer"
$DnsName = "demoContainer-$date"
$OsType = 'Linux'
$Port = '80'
$ContainerImage = 'nginx'

$containerGroupParams = @{
  'ResourceGroupName' = $ResourceGroup;
  'Name'              = $ContainerGroupName;
  'Image'             = $ContainerImage;
  'DnsNameLabel'      = $DnsName;
  'OsType'            = $OsType;
  'Port'              = $Port
}

New-AzureRmContainerGroup @containerGroupParams

Remove-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName
```

With it being that easy, I came up with the plan to use this as part of a TeamCity build job which would involve:

1. Authenticating to Azure
2. Creating the container
3. Running the tests against the container
4. Removing the container
5. Removing the Azure session

Because the build agents are running in AWS, I had to create a service principal to connect to Azure and run the PowerShell commands. This post will cover:

1. Creating a Service Principal with a Certificate
2. Giving the service principal just enough permissions to a resource group via a custom Azure role to create and remove containers
3. Setting up TeamCity build steps and parameters to connect to Azure, create, test against and remove then container the remove the Azure session
4. The PowerShell scripts to run in item 3.

The examples in this post was carried out using PowerShell Desktop Version 5.1 on Windows 10 1803.

## Azure Service Principal with Certificate

First I create the Azure Service Principal following the documentation. I wanted to use a certificate for the Service Principal authentication because I had not done this before.
Here's the code to create the Service Principal, it creates a certificate locally that is used for the Service Principal to connect. This certificate should be copied (included the private key) to where you want the service principal to be able to login (i.e. the build agents).

From https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell

```powershell
# Create an Azure Service principal with a cert for authentication
$certStoreLoc = 'cert:\CurrentUser\My'
$certSubject = 'CN=teamCityAzureContainerSP'
$servicePrincipalName = 'azure-container-instances-teamcity-testing'

# Create local self-signed cert - use cert authority in production
$cert = New-SelfSignedCertificate -CertStoreLocation $certStoreLoc -Subject $certSubject -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

Connect-AzureRMAccount

# Create the service principal with the certificate just created
$servicePrincipal = New-AzureRMADServicePrincipal -DisplayName $servicePrincipalName -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
Start-Sleep 20
```

![Creating the certificate in the terminal](/images/tc-azure-container-instances/create-cert.png)

![Creating the service principal in the terminal](/images/tc-azure-container-instances/create-sp.png)

This creates the AzureAD application and Service Principal the will be used by the TeamCity build step to authenticate to Azure.

Don't forget that you'll have to import he certificate to the build agent.

Export the cert with the private key

```powershell
$password = read-host -AsSecureString
Export-PfxCertificate -Cert $cert -FilePath C:\TEMP\cert-test.pfx -Password $password -Force
```

![Exporting the certificate from the certificate store](/images/tc-azure-container-instances/export-cert.png)

Import the cert using the password used to export it

```powershell
 # Local machine so build agent account can access it
 $password = Read-Host -AsSecureString
 Import-PfxCertificate -FilePath .\cert.pfx -Password $password -CertStoreLocation Cert:\LocalMachine\My\
```

![Importing the certificate to where the principal will run](/images/tc-azure-container-instances/import-cert.png)

## Azure custom role for Container Instances

Next step is to limit what the Service Principal can do in the Azure subscription(s). I like to limit to a specific resource group and will create a custom role that limits the Service Principal to be able to take actions on container instance groups within the resource group and nothing else.

First the policy below should be saved as a JSON document and in the Assingnable Scope property, update with the subscription ID. This policy allows the Principal to carry out any action against the container groups but could be locked down even further if required.

Update subscription-id-here with Azure SubscriptionID ```Get-AzureRmSubscription```

```json
{
  "Name": "Container Instance Container Group Manager",
  "Id": null,
  "IsCustom": true,
  "Description": "Allow full access to the Azure Container Instance container group resources",
  "Actions": [
    "Microsoft.ContainerInstance/containerGroups/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/subscription-id-here"
  ]
}
```

```powershell
New-AzureRmRoleDefinition -InputFile .\ContainerInstanceContainerGroupManagerRole.json
```

![Creating the Azure role in the terminal](/images/tc-azure-container-instances/create-role.png)

### How to find the namespace for roles

```powershell
Get-AzureRmProviderOperation | where-object {$_.Operation -like "Microsoft.ContainerInstance/containerGroups*"} | select-object -property operation
```

```powershell
Operation
---------
Microsoft.ContainerInstance/containerGroups/read
Microsoft.ContainerInstance/containerGroups/write
Microsoft.ContainerInstance/containerGroups/delete
Microsoft.ContainerInstance/containerGroups/restart/action
Microsoft.ContainerInstance/containerGroups/stop/action
Microsoft.ContainerInstance/containerGroups/start/action
Microsoft.ContainerInstance/containerGroups/containers/logs/read
```

https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles-powershell

https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations#microsoftcontainerinstance

## Assign roles to the Service Principal

Be patient after assign in the role, usually it takes place instantly but I have been caught out a couple of times when it took a little longer to propagate through. A good sign that it has propagated is if it shows up in the portal under the resource group IAM blade.

```powershell
# Apply custom role and reader role at the resource group level
$roleDefName = 'Container Instance Container Group Manager'
$resourceGroup = 'tc-containers'

New-AzureRmRoleAssignment -ObjectId $servicePrincipal.Id -RoleDefinitionName $roleDefName -ResourceGroupName $resourceGroup
New-AzureRmRoleAssignment -ObjectId $servicePrincipal.ApplicationId -RoleDefinitionName 'reader' -ResourceGroupName $resourceGroup
```

![Assign the custom Azure role](/images/tc-azure-container-instances/assign-role.png)

## PowerShell Scripts

Below are the PowerShell script that correspond with the TeamCity build steps. The parameters are passed through the scripts by TeamCity at build time.

The scripts and build do the following:

1. Authenticate with Azure using the sevice principal
2. Create a container instance group with the specified container
3. Run a simple test against the container
4. Remove the container
5. Remove the authenticated Azure session

1: Authenticate to Azure with the service principal.

```powershell
param(
  [Parameter(Mandatory = $true)]
  [String]
  $ApplicationId,
  [Parameter(Mandatory = $true)]
  [String]
  $TenantId,
  [Parameter(Mandatory = $true)]
  [String]
  $Thumbprint,
  [Parameter(Mandatory = $true)]
  [String]
  $ContextName
  
)

$authParams = @{
  'ServicePrincipal'      = $true;
  'CertificateThumbprint' = $Thumbprint;
  'ApplicationId'         = $ApplicationId;
  'Tenant'                = $TenantId;
  'ContextName'           = $ContextName
}

Connect-AzureRmAccount @authParams
```

2: Create the Container Group Instance

```powershell
param(
  [Parameter(Mandatory = $true)]
  [String]
  $ResourceGroup
)

$prefix = -join ((97..122) | Get-Random -Count 5 | ForEach-Object {[char]$_})
$date = Get-Date -Format yyyyMMddHHMMss
$ContainerGroupName = "tc-testing-containers-$date"
$DnsName = "$prefix-$date"
$OsType = 'Linux'
$Port = '80'
$ContainerImage = 'nginx'


$containerGroupParams = @{
  'ResourceGroupName' = $ResourceGroup;
  'Name'              = $ContainerGroupName;
  'Image'             = $ContainerImage;
  'DnsNameLabel'      = $DnsName;
  'OsType'            = $OsType;
  'Port'              = $Port
}

$containerGroup = New-AzureRmContainerGroup @containerGroupParams

while ((Get-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName).State -ne 'running') {
  Write-Output "Container state is: $((Get-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName).State)"
  Start-Sleep -Seconds 5
}

Write-Output "Container is: $((Get-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName).State)"

# Test that the after the container is running, wait until we can get a TCP connection on the container
while ((Test-NetConnection -ComputerName $containerGroup.Fqdn -Port $Port).TcpTestSucceeded -ne 'True') {
  Write-Output 'Waiting for Nginx'
  Start-Sleep -Seconds 5
}

# Update the TeamCity build parameters for use in later build steps
"##teamcity[setParameter name='containerGroupName' value='$ContainerGroupName']"
"##teamcity[setParameter name='containerUrl' value='$($containerGroup.Fqdn)']"
```

3: Run a simple test against the container

```powershell
param(
  [Parameter(Mandatory = $true)]
  [String]
  $ContainerUri
)

Write-Output $ContainerUri

$result = Invoke-WebRequest -UseBasicParsing -Uri $ContainerUri

Write-Output $result

if ($result.Content.Contains('Thank you for using nginx.')) {
  Write-Output 'Success, the website has the correct wording'
  Return 0
} else {
  Write-Output 'Failure, the wording on the website is wrong'
  Return 1
}
```

4: Remove the container instance group

```powershell
param(
  [Parameter(Mandatory = $true)]
  [String]
  $ResourceGroup,
  [Parameter(Mandatory = $true)]
  [String]
  $ContainerGroupName
)

Remove-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName
```

5: Remove the session

```powershell
param(
  [Parameter(Mandatory = $true)]
  [String]
  $ContextName
)

Disconnect-AzureRmAccount -ContextName $ContextName
```

## TeamCity Build Job

### Parameters

The Configuration Parameters configured are as follows:

1. applicationId - password - The ID of the Service Principal
2. containerGroup - string - Empty = updated by the PowerShell scripts using the service message to be used in later build steps
3. containerUrl - string - Empty = updated by the PowerShell scripts using the service message to be used in later build steps
4. contextName - string - Name of the context used for the authentication to Azure
5. resourceGroup - string - Name of the resource group the Service Principal has authorisation to create and remove container instances
6. tenantId - secure password - tenant ID of the Azure subscription
7. thumbprint - secure password - thumbprint of the certificate

![TeamCity build parameters](/images/tc-azure-container-instances/build-params.png)

### Build Step 1

1. Runner Type: PowerShell
2. Name: Authenticate to Azure
3. Execute step: If all previous steps finished successfully
4. PowerShell Version: left blank
5. Platform: Auto
6. Edition: Desktop
7. Format stderr output as: Warning
8. Working directory: Container-Instance/teamcity-build
9. Script: File
10. Script File: Container-Instance/teamcity-build/1AuthenticateToAzure.ps1
11. Script execution mode: Execute .ps1 file from external file
12. Script arguments:
    1. -ApplicationId:%applicationId%
    2. -TenantId:%tenantId%
    3. -Thumbprint:%thumbprint%
    4. -ContextName:%contextName%
13. Options: Add -NoProfile argument = checked
14. Additional command line parameters: blank
15. Run Step with docker container: blank

![TeamCity build step 1](/images/tc-azure-container-instances/build-step1.png)

The rest of the build steps have a similar setup as build 1, with different file paths etc. For brevity I will just list the script arguments of the following steps.

### Build Step 2

1. Script Arguments:
    * -ResourceGroup:%resourceGroup%

![TeamCity build step 2](/images/tc-azure-container-instances/build-step2.png)

### Build Step 3

1. Script Arguments:
    * -ContainerUri:%containerUrl%

![TeamCity build step 3](/images/tc-azure-container-instances/build-step3.png)

### Build Step 4

1. Script Arguments:
    * -ResourceGroup:%resourceGroup%
    * -ContainerGroupName:%containerGroupName%

![TeamCity build step 4](/images/tc-azure-container-instances/build-step4.png)

### Build Step 5

1. Script Arguments:
    * -ContextName:%contextName%

![TeamCity build step 5](/images/tc-azure-container-instances/build-step5.png)

This [blog post] was extremely helpful in working out how to pass the build parameters to the PowerShell scripts.

## Summary

Azure Container Instances make it really easy to spin up a container without the overhead of setting up and managing the underlying hardware. The service is create for this use case of spinning up a container and running tests. This example showed how to do it in TeamCity but it would be just as simple to use Jenkins. If the TeamCity build agents were running in Azure then instead of creating the service principal, managed identities can be used. The service principal allows you to take advantage of container instances if the TeamCity server is running on-prem or elsewhere which gives great flexibility. I really like the container instance service and it's a great addition to the resources offered by Microsoft on Azure.

[hyper-v to be installed]: https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/linux-containers
[Azure Container Instances docs]: https://docs.microsoft.com/en-us/azure/container-instances/container-instances-container-groups
[channel9 video on Azure container instances]: https://channel9.msdn.com/Shows/Tuesdays-With-Corey/More-info-on-Azure-Container-Instances
[PowerShell section]: https://docs.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-powershell
[blog post]: http://ralbu.com/teamcity-build-parameters-for-powershell
[managed identities]: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview