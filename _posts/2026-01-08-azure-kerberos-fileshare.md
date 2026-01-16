---
title: "How to Use Azure Files with Entra ID Kerberos Authentication (No Azure AD DS)"
date: 2027-01-08
categories:
  - Azure
  - File Sharing
  - Authentication
tags:
  - Azure Files
  - Kerberos
  - Microsoft Entra ID
  - Intune
  - PowerShell
---

Azure Files now supports **Kerberos authentication using Microsoft Entra ID**, eliminating the need for Azure AD Domain Services or on-premises Active Directory. This allows Azure AD–joined devices to securely access SMB file shares using native Kerberos authentication.

For more details and step-by-step instructions, see Microsoft's documentation: [Enable hybrid identities for Azure Files Kerberos authentication](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-identity-auth-hybrid-identities-enable).

In this post, I will walk through **how to configure Azure Files with Entra ID Kerberos authentication**, assign permissions, and mount the file share from user devices.  
For client-side automation and deployment, this setup aligns well with the approach described in the [SharePoint to Azure Files migration post](../2025-12-22-sharepointtoazurefiles.md).

Screenshots and diagrams will be added in a future update.

---

## Prerequisites

Before starting, ensure you have:

- An Azure subscription
- Microsoft Entra ID
- Azure AD–joined Windows devices
- Microsoft Intune
- Azure PowerShell (`Az` module)

---
![Storage Account](/assets/img/blog/azure_kerb/mynd1.png)

## Step 1: Create an Azure Storage Account


Azure Files with Kerberos authentication requires a **StorageV2** account.

```powershell
# Variables
$resourceGroup = "rg-azurefiles-kerberos"
$location      = "westeurope"
$storageName   = "stazurefileskerb01"
```

# Create resource group
New-AzResourceGroup `
  -Name $resourceGroup `
  -Location $location

# Create storage account
New-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind StorageV2 `
  -EnableLargeFileShare

## Step 2: Create an Azure File Share
```powershell
# Get storage account context
$storageAccount = Get-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName

$ctx = $storageAccount.Context

# Create file share
New-AzStorageShare `
  -Name "profiles" `
  -Context $ctx `
  -QuotaGiB 1024
```

## Step 3: Enable Entra ID Kerberos Authentication
This step enables Kerberos authentication without Azure AD DS.

```powershell
Set-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -EnableAzureActiveDirectoryKerberosForFile $true

# Verify the configuration:
Get-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName |
Select-Object EnableAzureActiveDirectoryKerberosForFile
```

## Step 4: Assign Permissions Using Azure RBAC
Azure Files uses Azure RBAC when Kerberos authentication is enabled.

```powershell
# Variables
$shareName = "profiles"
$groupName = "AzureFiles-Users"

# Get Entra ID group
$group = Get-AzADGroup -DisplayName $groupName

# Assign SMB permissions
New-AzRoleAssignment `
  -ObjectId $group.Id `
  -RoleDefinitionName "Storage File Data SMB Share Contributor" `
  -Scope "/subscriptions/$((Get-AzContext).Subscription.Id)/resourceGroups/$resourceGroup/providers/Microsoft.Storage/storageAccounts/$storageName/fileServices/default/fileshares/$shareName"
```

| Role                                             | Use case     |
| ------------------------------------------------ | ------------ |
| Storage File Data SMB Share Reader               | Read-only    |
| Storage File Data SMB Share Contributor          | Read/write   |
| Storage File Data SMB Share Elevated Contributor | Full control |

## Step 5: Intune Integration
pictures
pictures
pictures
pictures
pictures
pictures
pictures
pictures

## Step 6: Mount the File Share from User Devices
Once configured, users can mount the file share without providing credentials.

```powershell
New-PSDrive `
  -Name Z `
  -PSProvider FileSystem `
  -Root "\\stazurefileskerb01.file.core.windows.net\profiles" `
  -Persist
```
Sometimes the old kerberos ticket needs to be purged before yoy can mount the drive
klist purge

Wolla,, should have netdrive with you old sharepoint data mounted.