---
title: MS Graph API - Get Secure Score for tenant
date: 2023-03-25 22:49:00 -0000
categories: [Azure,MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---
# Secure Score for O365 Tenant 
## Getting the secure score for a tenant is a simple task.
You only need to call the following endpoint: https://graph.microsoft.com/beta/security/secureScores
and divide  the result by 100 to get the percentage.

```powershell
# Needs permission SecurityEvents.Read.All

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

$path = "C:\temp\"

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
$uri = "https://graph.microsoft.com/beta/security/secureScores"

$Result = @()
[array]$Response = Get-GraphData -AccessToken $Token -Uri $uri

if ($Response) {
    ForEach ($Respons in $Response) {
      $SecureScore = $Respons.currentScore / $Respons.maxScore * 100
  
      $Result += New-Object PSObject -property $([ordered]@{ 
          CreatedDateTime   = $Respons.createdDateTime
          CurrentScore      = $Respons.currentScore
          MaxScore          = $Respons.maxScore
          LicensedUserCount = $Respons.licensedUserCount
          ActiveUserCount   = $Respons.activeUserCount
          SecureScore       = [math]::Round($SecureScore, 2)
        })
    }
 
  }
else {
    Write-Host "No SecureScore data found"
}

# export to csv
$Result | export-csv -path "$path\SecureScore_over_time.csv" -NoTypeInformation
$Response | Select-Object -ExpandProperty averageComparativeScores -First 1 | export-csv "$path\averageComparativeScores.csv" -NoTypeInformation
$Response | Select-Object -ExpandProperty enabledServices -First 1 | Out-File $path\enabledServices.csv
$Response | Select-Object -ExpandProperty controlScores -First 1 | Select-Object controlName, IsApplicable, scoreInPercentage, implementationStatus | Out-File $path\controlScores.csv
$Response | Select-Object -ExpandProperty controlScores -First 1 | Select-Object controlName, description | Out-File $path\controlScores_info.csv

#list many csv in path to one variable
$files = Get-ChildItem -Path C:\temp\ -Filter *.csv -Recurse | Select-Object -ExpandProperty FullName
```
