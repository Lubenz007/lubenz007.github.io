---
title: Getting licenses assigned to user accounts
date: 2023-04-01 00:27:00 -0000
categories: [Azure,MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---

# Create a report of licenses assigned to Azure AD user accounts using the Microsoft Graph API
This is a rewrite from [here](https://practical365.com/create-licensing-report-microsoft365-tenant/) practical365.com
> My misson was to rewrite the script to use the Microsoft Graph API instead of the Azure AD PowerShell module.
{: .prompt-info }

```powershell

$ClientID = ''
$ClientSecret = ''
$tenant_Id = ''

# Connect to Graph #

$Body = @{    
    Grant_Type    = "client_credentials"
    resource      = "https://graph.microsoft.com"
    client_id     = $clientId
    client_secret = $clientSecret
} 
  
$ConnectGraph = Invoke-RestMethod -Uri "https://login.microsoft.com/$tenant_Id/oauth2/token?api-version=1.0" -Method POST -Body $Body

# Variable Collections #

$CSVOutputFile = "c:\temp\Microsoft365LicensesReport.CSV"
$today = Get-Date

$Headers = @{
    'Content-Type'  = "application/json"
    'Authorization' = "Bearer $($ConnectGraph.access_token)"
}

$token = $ConnectGraph.access_token

# Force TLS 1.2.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

function Get-GraphData {
    param (
        [parameter(Mandatory)]
        [string]$AccessToken,
      
        [parameter(Mandatory)]
        [string]$Uri
    )

    $Headers = @{
        'Authorization' = "Bearer $AccessToken"
    }

    do {
        $Results = Invoke-RestMethod -Uri $Uri -Headers $Headers -ErrorAction Stop

        $QueryResults += $Results.value

        $Uri = $Results.'@odata.nextLink'
    } while ($Uri)

    return $QueryResults
}

#This request get SecureScore

[array]$Skus = Get-GraphData -AccessToken $Token -Uri "https://graph.microsoft.com/beta/subscribedSkus"
#[Array]$Skus = Get-MgSubscribedSku

# Generate CSV of all product SKUs used in tenant
[array]$Sku = $Skus | Select-Object SkuId, SkuPartNumber
# Generate list of all service plans used in SKUs in tenant
$SPData = [System.Collections.Generic.List[Object]]::new()
ForEach ($Sk in $Skus) {
    ForEach ($SP in $Sk.ServicePlans) {
        $SPLine = [PSCustomObject][Ordered]@{  
            ServicePlanId          = $SP.ServicePlanId
            ServicePlanName        = $SP.ServicePlanName
            ServicePlanDisplayName = $SP.ServicePlanName 
        }
        $SPData.Add($SPLine)
    }
}

$SkuHashTable = @{}
ForEach ($Line in $Sku) { 
    $SkuHashTable.Add([string]$Line.SkuId, [string]$Line.skuPartNumber) 
}
$ServicePlanHashTable = @{}
ForEach ($Line2 in $SPData) { $ServicePlanHashTable.Add([string]$Line2.ServicePlanId, [string]$Line2.ServicePlanDisplayName) }

[Array]$Users = Get-GraphData -AccessToken $Token -Uri "https://graph.microsoft.com/beta/users"
$Report = [System.Collections.Generic.List[Object]]::new()

ForEach ($User in $users) {
    If ([string]::IsNullOrWhiteSpace($User.AssignedLicenses) -eq $true) {
       # Only process account if it has some licenses
       Write-Host "Processing" $User.DisplayName
       [array]$LicenseInfo = $Null; [array]$DisabledPlans = $Null
       ForEach ($License in $User.AssignedLicenses) {
          If ($SkuHashTable.ContainsKey($License.SkuId) -eq $True) {
             # We found a match in the SKU hash table
             $LicenseInfo += $SkuHashTable.Item($License.SkuId) 
          }
          Else {
             # Nothing doing, so output the SkuID
             $LicenseInfo += $License 
          }
          # Report any disabled service plans in licenses
          If ([string]::IsNullOrWhiteSpace($License.DisabledPlans) -eq $False ) {
             # Check if disabled service plans in a license
             ForEach ($DisabledPlan in $License.DisabledPlans) {
                # Try and find what service plan is disabled
                If ($ServicePlanHashTable.ContainsKey($DisabledPlan) -eq $True) {
                   # We found a match in the Service Plans hash table
                   $DisabledPlans += $ServicePlanHashTable.Item($DisabledPlan) 
                }
                Else {
                   # Nothing doing, so output the Service Plan ID
                   $DisabledPlans += $DisabledPlan 
                }
             } # End ForEach disabled plans
          } # End if check for disabled plans  
       } # End Foreach Licenses
       # Report information
       [string]$DisabledPlans = $DisabledPlans -join ", "
       [string]$LicenseInfo = $LicenseInfo -join (", ")
       $ReportLine = [PSCustomObject][Ordered]@{  
          User             = $User.DisplayName
          UPN              = $User.UserPrincipalName
          Country          = $User.Country
          Department       = $User.Department
          Title            = $User.JobTitle
          Licenses         = $LicenseInfo
          "Disabled Plans" = $DisabledPlans 
       }
       Write-Host $ReportLine
       $Report.Add($ReportLine)
    } #end If account is licensed
    Else { $UnlicensedAccounts++ }
 } # End ForEach Users
 $Report | Export-CSV -NoTypeInformation $CSVOutputFile


```

![Desktop View](/assets/img/blog/license.png){: .normal-image}