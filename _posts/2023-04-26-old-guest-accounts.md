---
title: Getting old guest accounts in Azure AD
date: 2023-04-26 23:00:49:00 -0000
categories: [MsGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---

This is a rewrite from [here](https://office365itpros.com/2019/10/15/report-old-guest-accounts/) office365itpros.com
I have added some more properties to the report.
And use msgraph instead of mggraph.

```powershell
# Needs permission User.Read.All

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
#This request get users list with signInActivity.
$uri = "https://graph.microsoft.com/beta/users"
 
$Result = @()
[array]$Response = Get-GraphData -AccessToken $Token -Uri $uri

if ($Response) {
  ForEach ($Respons in $Response) {
      $Result += New-Object PSObject -property $([ordered]@{ 
      DisplayName = $Respons.displayName
      UserPrincipalName = $Respons.userPrincipalName
      UsageLocation = $Respons.usageLocation
      Contry = $Respons.country
      LastSignInDateTime = if($Respons.signInActivity.lastSignInDateTime) { [DateTime]$Respons.signInActivity.lastSignInDateTime } Else {$null}
      IsLicensed  = if ($Respons.assignedLicenses.Count -ne 0) { $true } else { $false }
      IsGuestUser  = if ($Respons.userType -eq 'Guest') { $true } else { $false }
      RefreshTokensValidFromDateTime = $Respons.RefreshTokensValidFromDateTime
      onPremisesDistinguishedName = $Respons.onPremisesDistinguishedName
  
      })
  }

}
else {
  Write-Host "No User data found"
}

$GuestUsers = $Result | Where-Object{$_.IsGuestUser -eq "TRUE"}

$GuestAccountAge = 365 # Value used for guest age comparison. If you want this to be a different value (like 30 days), change this here.
#$GuestUsers = $users.value -All $true -Filter "UserType eq 'Guest'" | Sort DisplayName
$Today = (Get-Date); $StaleGuests = 0
$Report = [System.Collections.Generic.List[Object]]::new()
# Check each account and find those over 365 days old
ForEach ($Guest in $GuestUsers) {
   $AADAccountAge = ($Guest.RefreshTokensValidFromDateTime | New-TimeSpan).Days
   If ($AADAccountAge -gt $GuestAccountAge) {
      $StaleGuests++
      #Write-Host "Processing" $Guest.DisplayName
      $i = 0; $GroupNames = $Null
      # Find what Microsoft 365 Groups the guest belongs to... if any
      $ReportLine = [PSCustomObject]@{
           UPN     = $Guest.UserPrincipalName
           Name    = $Guest.DisplayName
           Age     = $AADAccountAge
           Created = $Guest.RefreshTokensValidFromDateTime  
           }      
     $Report.Add($ReportLine) }
}
# Output the report
$Report | Sort Age -Descending | Format-Table -AutoSize
Write-Host "Found" $StaleGuests "stale guest accounts."

#reference: https://office365itpros.com/2019/10/15/report-old-guest-accounts/

```
