---
title: MS Graph API - Get User info
date: 2023-03-25 13:49:00 -0000
categories: [Azure,MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---
## Microsoft Graph API can be used to get the last logon time of a user. To do this, you will need to make a GET request to the /users/{id}/lastLogonTimeStamp endpoint. This endpoint will return the last logon time of the user in the form of a timestamp. You can then use this timestamp to determine the exact date and time of the user's last logon.

This query get's.
> DisplayName,UserPrincipalName,UsageLocation,Contry,LastSignInDateTime,IsLicensed,IsGuestUser
{: .prompt-info }

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

<table>
  {% for row in site.data.lastlogon %}
    {% if forloop.first %}
    <tr>
      {% for pair in row %}
        <th>{{ pair[0] }}</th>
      {% endfor %}
    </tr>
    {% endif %}
    {% tablerow pair in row %}
      {{ pair[1] }}
    {% endtablerow %}
  {% endfor %}
</table>
