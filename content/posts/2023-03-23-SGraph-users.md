---
title: MS Graph API - Get User info
date: 2023-03-25 13:49:00 -0000
categories: [MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
summary: "Learn how to use Microsoft Graph API to retrieve user information including last sign-in times, licensing status, and user properties with PowerShell."
---

Microsoft Graph API can be used to get comprehensive user information including the last logon time. This guide shows how to query user data including DisplayName, UserPrincipalName, UsageLocation, Country, LastSignInDateTime, IsLicensed, and IsGuestUser.

<!--more-->

## What This Query Retrieves

- DisplayName, UserPrincipalName, UsageLocation, Country
- LastSignInDateTime, IsLicensed, IsGuestUser
- Complete user activity and licensing information

```powershell
#client_id and client_secret are generated in Azure AD
$ClientID = ''
$ClientSecret = ''
$tenant_Id = ''

# Create the body of the request.
$Body = @{    
    Grant_Type    = "client_credentials"
    resource      = "https://graph.microsoft.com"
    client_id     = $clientId
    client_secret = $clientSecret
} 

# Get the access token.
$ConnectGraph = Invoke-RestMethod -Uri "https://login.microsoft.com/$tenant_Id/oauth2/token?api-version=1.0" -Method POST -Body $Body

# Force TLS 1.2.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

# Get the access token.
$token = $ConnectGraph.access_token

# Variable Collections #  
$Result = @()

#This request get users list with signInActivity.
$Uri = "https://graph.microsoft.com/beta/users?$select=displayName,userPrincipalName,contry,UsageLocation,userType,assignedLicenses,signInActivity,lastSignInDateTime&$top=999"

#function to get graph data with pagination
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

# Get the users.
[array]$Users = Get-GraphData -AccessToken $Token -Uri $uri

# Loop through the results and add them to the output array.
ForEach ($User in $Users) {
 
    $Result += New-Object PSObject -property $([ordered]@{ 
            DisplayName        = $User.displayName
            UserPrincipalName  = $User.userPrincipalName
            UsageLocation      = $user.usageLocation
            Contry             = $User.country
            LastSignInDateTime = if ($User.signInActivity.lastSignInDateTime) { [DateTime]$User.signInActivity.lastSignInDateTime } Else { $null }
            IsLicensed         = if ($User.assignedLicenses.Count -ne 0) { $true } else { $false }
            IsGuestUser        = if ($User.userType -eq 'Guest') { $true } else { $false }
        })
}
 
# Write the results to a CSV file.
$Logfile = "lastlogon.csv"
$LogItem = New-Item -ItemType File -Name $Logfile
$Result | ConvertTo-Csv | Out-File $LogItem -Append

```

