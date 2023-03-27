---
title: Check Registered Apps in Azure AD
date: 2023-03-27 22:15:00 -0000
categories: [Azure,MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---

# Check Registered Apps in Azure AD
This script will check all registered apps in Azure AD and will return the following information:
> CredentialType, DisplayName,AppId,ExpiryDate,StartDate,Type,Usage,Owners,Expired
{: .prompt-info }

```powershell
# needs permission Application.Read.All,Directory.Read.All

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
$uri = "https://graph.microsoft.com/beta/applications/"

[array]$apps = Get-GraphData -AccessToken $Token -Uri $uri

$credentials = @()

if ($apps) {
    foreach ($app in $apps) {
        $ApiUrl = "https://graph.microsoft.com/beta/applications/"+$app.id+"/owners"
        $owner = Invoke-WebRequest -Method GET -Uri $ApiUrl -ContentType "application/json" -Headers $headers | ConvertFrom-Json

        $app.KeyCredentials | foreach-object {
            #write-host $_.KeyId $_.DisplayName
            $credentials += [PSCustomObject] @{
                CredentialType = "KeyCredentials";
                DisplayName    = $app.DisplayName;
                AppId          = $app.AppId;
                ExpiryDate     = $_.EndDateTime;
                StartDate      = $_.StartDateTime;
                #KeyID = $_.KeyId;
                Type           = $_.Type;
                Usage          = $_.Usage;
                Owners         = $owner.value.userPrincipalName;
                Expired        = (([DateTime]$_.EndDateTime) -lt $today) ? "Yes" : "No";
            }
        }

        $app.PasswordCredentials | foreach-object {
            #write-host $_.KeyId $_.DisplayName
            $credentials += [PSCustomObject] @{
                CredentialType = "PasswordCredentials";
                DisplayName    = $app.DisplayName;
                AppId          = $app.AppId;
                ExpiryDate     = $_.EndDateTime;
                StartDate      = $_.StartDateTime;
                #KeyID = $_.KeyId;
                Type           = 'NA';
                Usage          = 'NA';
                Owners         = $owner.value.userPrincipalName;
                Expired        = (([DateTime]$_.EndDateTime) -lt $today) ? "Yes" : "No";
            }
        }
    }
}
$credentials | Export-Csv -Path $path\credentials.csv -NoTypeInformation
```