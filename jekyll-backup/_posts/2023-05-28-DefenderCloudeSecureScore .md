---
title: Get Azure Defender for Cloud score Via API
date: 2023-05-28 14:00:00 -0000
categories: [Security]
tags: [azure,security ]     # TAG names should always be lowercase
---

How to build a report of Azure Defender for Cloud score Via API, and export it to CSV.
Why do we need this? share it with your ?, and show them the progress of your security posture.


>Need to create service principal with the following permissions: Read RBAC on each subscription.
{: .prompt-info }

```powershell
#set the variables
$ClientID = ''
$ClientSecret = ''
$tenant_Id = ''

# your subscription Ids
$subscription_Ids = ''

# Create the body of the request.
$Body = @{    
    Grant_Type    = "client_credentials"
    Resource      = "https://management.azure.com/"
    client_id     = $clientId
    client_secret = $clientSecret
}

$ConnectGraph = Invoke-RestMethod -Uri "https://login.microsoft.com/$tenant_Id/oauth2/token?api-version=1.0" -Method POST -Body $Body

$Headers = @{
    'Content-Type'  = "application/json"
    'Authorization' = "Bearer $($ConnectGraph.access_token)"
}

# Force TLS 1.2.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

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

# Get the data
foreach ($subscription_Id in $subscription_Ids) {
    $uri1 = "https://management.azure.com/subscriptions/$subscription_Id/providers/Microsoft.Security/secureScores?api-version=2020-01-01"    
    [array]$secureScores = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri1
    $uri2 = "https://management.azure.com/subscriptions/$subscription_Id/providers/Microsoft.Security/secureScores/ascScore/securescorecontrols?api-version=2020-01-01"
    [array]$ascScores = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri2
    $uri3 = "https://management.azure.com/subscriptions/$subscription_Id/providers/Microsoft.Security/secureScoreControlDefinitions?api-version=2020-01-01"
    [array]$secureScoreControlDefinitions = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri3
    $uri4 = "https://management.azure.com/subscriptions/$subscription_Id/providers/Microsoft.Security/assessments?api-version=2020-01-01"
    [array]$assessments = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri4
    $uri5 = "https://management.azure.com/subscriptions/$subscription_Id/providers/Microsoft.Security/secureScores/ascScore/securescorecontrols?api-version=2020-01-01&expand=definition"
    [array]$securescorecontrols = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri5
}

# Create the report
$ReportLineScore = [PSCustomObject][Ordered]@{  
    MaxScore   = $secureScores.Properties.score.max
    Current    = $secureScores.Properties.score.current
    percentage = $secureScores.Properties.score.percentage * 100 
}

$SPData = [System.Collections.Generic.List[Object]]::new()
ForEach ($ascScore in $ascScores) {
    $SPLine = [PSCustomObject][Ordered]@{  
        Name          = $ascScore.properties.displayName
        Healthy       = $ascScore.properties.healthyResourceCount
        UnHealthy     = $ascScore.properties.unhealthyResourceCount
        NotApplicable = $ascScore.properties.notApplicableResourceCount
        weight        = $ascScore.properties.weight
        maxScore      = $ascScore.properties.score.max
        currentScore  = $ascScore.properties.score.current
        percentage    = $ascScore.properties.score.percentage * 100
    }
    $SPData.Add($SPLine)
}

$SPData1 = [System.Collections.Generic.List[Object]]::new()
foreach ($secureScoreControlDefinition in $secureScoreControlDefinitions) {
    $SPLine1 = [PSCustomObject][Ordered]@{  
        Name     = $secureScoreControlDefinition.properties.displayName
        MaxScore = $secureScoreControlDefinition.properties.maxScore
    }
    $SPData1.Add($SPLine1)
}

$SPData2 = [System.Collections.Generic.List[Object]]::new()
foreach ($assessment in $assessments) {
    $SPLine2 = [PSCustomObject][Ordered]@{  
        Name     = $assessment.properties.displayName
        Status   = $assessment.properties.status.description
        Code     = $assessment.properties.status.code
        Cause    = $assessment.properties.status.cause
        Details  = $assessment.properties.resourceDetails.Id
    }
    $SPData2.Add($SPLine2)
}

$SPData3 = [System.Collections.Generic.List[Object]]::new()
foreach ($securescorecontrol in $securescorecontrols) {
    $SPLine3 = [PSCustomObject][Ordered]@{  
        Name               = $securescorecontrol.properties.displayName
        HealthyCount       = $securescorecontrol.properties.healthyResourceCount
        UnHealthyCount     = $securescorecontrol.properties.unhealthyResourceCount
        NotApplicableCount = $securescorecontrol.properties.notApplicableResourceCount
        
    }
    $SPData3.Add($SPLine3)
}

$ReportLineScore | Export-Csv -Path "c:\temp\ReportLineScore.csv" -NoTypeInformation -Encoding UTF8
$SPData | Export-Csv -Path "c:\temp\SPData.csv" -NoTypeInformation -Encoding UTF8
$SPData1 | Export-Csv -Path "c:\temp\SPData1.csv" -NoTypeInformation -Encoding UTF8
$SPData2 | Export-Csv -Path "c:\temp\SPData2.csv" -NoTypeInformation -Encoding UTF8
$SPData3 | Export-Csv -Path "c:\temp\SPData3.csv" -NoTypeInformation -Encoding UTF8

```

|MaxScore|Current|percentage|
|:--|:--|:--|
|31|24|77,42|

|Name|MaxScore|
|:--|:--|
|Protect applications against DDoS attacks|2|
|Enable MFA|10|
|Encrypt data in transit|4|
|Restrict unauthorized network access|4|
|Implement security best practices|0|
|Apply adaptive application control|3|
|Enable auditing and logging|1|
|Enable encryption at rest|4|
|Enable endpoint protection|2|
|Apply system updates|6|
|Manage access and permissions|4|
|Remediate security configurations|4|
|Secure management ports|8|
|Remediate vulnerabilities|6|
|Enable enhanced security features|0|