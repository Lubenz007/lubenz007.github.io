---
title: Monitor Azure Usage for a subscription 
date: 2023-05-13 14:00:00 -0000
categories: [Usage]
tags: [azure,usage]     # TAG names should always be lowercase
---

Why do we need to monitor Azure usage? Well, if you have a subscription with a spending limit, you need to keep track of your usage. If you go over the spending limit, your resources will be disabled. And you will need to pay for the overage.

> **Info:** My concern is that some one gets a hold of my subscription, and start to use it. And I will get a big bill at the end of the month.

Using Get-AzConsumptionUsageDetail to get the usage for a subscription. Works not for Azure Plan subscriptions.
Bcause of this error: ((400) Subscription scope usage is not supported for current api version. Please use api version after 2019-10-01)

> **Tip:** But we can use the Microsoft.Consumption/usageDetails API to get the usage for a subscription.

First we need to create a Service Principal, and we need to give service principal billing role on the subscription.


```powershell
#This script will get the usage for a subscription

$startDate = (Get-Date).AddDays(-2).ToString("yyyy-MM-dd")
$endDate = (Get-Date).AddDays(-1).ToString("yyyy-MM-dd")

#set the variables
$ClientID = ''
$ClientSecret = ''
$tenant_Id = ''
$subscriptionId = ''

# Set the URI for the request.
$uri1 = "https://management.azure.com/subscriptions/$subscriptionId/providers/Microsoft.Consumption/usageDetails?api-version=2023-03-01&startDate=${startDate}&endDate=${startDate}"
$uri2 = "https://management.azure.com/subscriptions/$subscriptionId/providers/Microsoft.Consumption/usageDetails?api-version=2023-03-01&startDate=${endDate}&endDate=${endDate}"


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

#$useage = Invoke-WebRequest -Method GET -Uri $Uri -ContentType "application/json" -Headers $headers | ConvertFrom-Json
$uritwodaysago = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri1
$uriyesterday = Get-GraphData -AccessToken $ConnectGraph.access_token -Uri $uri2

$Resultstwodaysago = $uritwodaysago | Select-Object -ExpandProperty properties | Measure-Object -Property costInBillingCurrency -Sum | Select-Object Sum
$Resultyesterday = $uriyesterday | Select-Object -ExpandProperty properties | Measure-Object -Property costInBillingCurrency -Sum | Select-Object Sum

$Resultstwodaysago = [math]::Floor($Resultstwodaysago.sum)
$Resultyesterday = [math]::ceiling($Resultyesterday.sum)
$percentof = [math]::Floor($Resultstwodaysago * 1.4)

if ($Resultstwodaysago -le $percentof -and $Resultyesterday -lt 100) {
    Write-Host "OK: Yesterday's Azure spending ($Resultyesterday Euro) is not 40% more than 2 days ago ($Resultstwodaysago Euro) and not more than 100 Euro | yesterday=$Resultyesterday, spending2daysago=$Resultstwodaysago"
}
else {
    Write-Host  "Alarm!! Tom Much Azure Spending"
    $webhookUri = "webhook from teams"
    $body = @{
        "@context"   = "http://schema.org/extensions"
        "@typecod"   = "MessageCard"
        "themeColor" = "d70000"
        "title"      = "Alarm!! To Much Azure Spending"
        "text"       = "Some thing is wrong in Azure. Yesterday's Azure spending ($Resultyesterday Euro) is 40% more than 2 days ago ($Resultstwodaysago Euro) or more than 100 Euro"
    }
    Invoke-RestMethod -Uri $webhookUri -Method Post -Body (ConvertTo-Json -InputObject $body) 
}
  
```