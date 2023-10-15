---
title: Arc - Diskspace Monitoring - Teams)
date: 2023-10-15 02:00:00 -0000
categories: [Azure,Monitor]
tags: [azure,monitor]     # TAG names should always be lowercase
---

- Upon successfully installing Azure Arc on your Server and activating Insights, you open the door to a powerful monitoring capability. One essential aspect is overseeing disk space usage on your server, ensuring optimal performance and preventing potential issues. This example provides a step-by-step guide on how to set up disk space monitoring and receive instant notifications via Microsoft Teams when space is running low.

- Furthermore, this versatile approach can be extended to send email alerts, providing an additional layer of flexibility in how you manage critical resource thresholds. This comprehensive monitoring strategy empowers you with the tools to proactively maintain your server's health and responsiveness, bolstering the reliability of your operations.

- The query below is a simple example of how to monitor disk space usage on your server. It returns the percentage of free space on the C: drive, which is then used to trigger an alert when the threshold is exceeded. The query can be customized to monitor other drives on your server, and the threshold can be adjusted to suit your needs.

'''powershell
InsightsMetrics 
| where Name == "FreeSpaceMB" 
| summarize arg_max(TimeGenerated, *) by Tags, Computer 
| extend Drive = tostring(parse_json(Tags)["vm.azm.ms/mountId"]) 
| extend Size = toreal(parse_json(Tags)["vm.azm.ms/diskSizeMB"]) 
| project TimeGenerated, Computer, Drive, bin(SizeGB = Size / 1024, 0.1), bin(FreeGB = Val / 1024, 1) 
| join kind=inner (InsightsMetrics 
 | where Name == "FreeSpacePercentage" 
 | summarize arg_max(TimeGenerated, *) by Tags, Computer 
 | extend Drive = tostring(parse_json(Tags)["vm.azm.ms/mountId"]) 
 | project TimeGenerated, Computer, Drive, bin(FreePercent = Val, 1.1))on Computer, Drive 
| project TimeGenerated, Computer, Drive, SizeGB, FreeGB, FreePercent 
| where FreePercent <= 90.0 //
//| where FreeGB <= 10.0 
| order by Computer asc
'''
![Create Alert from Azure Monitor](AzureMonitor.png)

![Create an alert rule](Create_an_alert_rule.png)


- We need to enable webhooks in Microsoft Teams and copy the webhook URL. Finally, we need to create Azure automation runbook and configure it to send notifications to the channel.
We also need to enable the Azure Automation account to access the Log Analytics workspace. To do this, we need to give automation account read access to the Log Analytics workspace and resource group where the arc servers are located.


'''powershell
 # So we are going to use the same query as in Azure Monitor to get the results and send it to Teams

 # Connect to Azure for Log Analytics
Connect-AzAccount -Identity
$context = Set-AzContext -Subscription "benedikt.egilsson – MPN"
$workspaceName = "arc-servers"
$workspaceRG = "arc-servers"
$WorkspaceID = (Get-AzOperationalInsightsWorkspace -Name $workspaceName -ResourceGroupName $workspaceRG).CustomerID

$query = "InsightsMetrics
| where Name == `"FreeSpaceMB`"
| summarize arg_max(TimeGenerated, *) by Tags, Computer 
| extend Drive = tostring(parse_json(Tags)[`"vm.azm.ms/mountId`"]) 
| extend Size = toreal(parse_json(Tags)[`"vm.azm.ms/diskSizeMB`"]) 
| project TimeGenerated, Computer, Drive, bin(SizeGB = Size / 1024, 0.1), bin(FreeGB = Val / 1024, 1) 
| join kind=inner (InsightsMetrics 
 | where Name == `"FreeSpacePercentage`" 
 | summarize arg_max(TimeGenerated, *) by Tags, Computer 
 | extend Drive = tostring(parse_json(Tags)[`"vm.azm.ms/mountId`"]) 
 | project TimeGenerated, Computer, Drive, bin(FreePercent = Val, 1.1))on Computer, Drive 
| project TimeGenerated, Computer, Drive, SizeGB, FreeGB, FreePercent 
| where FreePercent <= 90.0 //
//| where FreeGB <= 10.0 
| order by Computer asc"

$kqlQuery = Invoke-AzOperationalInsightsQuery -WorkspaceId $WorkspaceID -Query $query

$webhookUri = "your webhook url"

 # Define the message as a PowerShell object

$message = @{
    context   = "http://schema.org/extensions"
    type      = "MessageCard"
    themeColor = "d70000"
    title      = "Machine with low disk space"
    text = "$($kqlQuery.results | Format-List | Out-String)"
}

# Convert the message to JSON
$jsonMessage = $message | ConvertTo-Json

# Send the message using Invoke-RestMethod
Invoke-RestMethod -Method Post -ContentType 'application/json' -Body $jsonMessage -Uri $webhookUri
'''

![Disk Alert Teams](diskalert_teams.png)
