---
title: Upgrade Hyper-V Hosts with Azure Arc
date: 2023-04-04 22:15:00 -0000
categories: [HyperV,Automation]
tags: [hyperv,automation]     # TAG names should always be lowercase
---

Azure Arc is a powerful tool for managing and updating hybrid infrastructure, including standalone Hyper-V hosts. Demo how to use Azure Arc to update a standalone Hyper-V host, using a scheduled update task in Automation Account and pre and post scripts that shut down and start up VMs.

Pre-requisites

```powershell
# Stop all running VMs on the Hyper-V host and wait for them to shut down
# save the names of the running VMs to a file
$filePath = "C:\temp\runningvm.txt"
(Get-vm | Where-Object { $_.State -eq "Running" }).name | Out-File $filePath

$runningvm = Get-Content $filePath

foreach ($Name in $runningvm) {
    Stop-VM $Name

    do {
        $VM1 = get-vm -Name $Name
        Write-Progress -Activity "Waiting for the VM to shutdown" 
    } until ($Null -eq $VM1.Heartbeat)

}
# send a webhook to Teams to notify that the VMs have been shut down
$webhookUri = ""
$body = @{
"@context" = "http://schema.org/extensions"
"@type" = "MessageCard"
"themeColor" = "d70000"
"title" = "Send Webhook to Teams"
"text" = "This is a message sent from Powershell"
}
Invoke-RestMethod -Uri $webhookUri -Method Post -Body (ConvertTo-Json -InputObject $body) 
```

Post-requisites

```powershell
# Start up all VMs that were shut down during the update process
$filePath = "C:\temp\runningvm.txt"
$runningvm = Get-Content $filePath

foreach ($Name in $runningvm) {
    Start-VM $Name
    do {
        $network = get-vm -name $name | get-VMNetworkAdapter
        Write-Progress -Activity "Waiting for VM Network" 
    } until ($network.Status -eq "ok") 
# Wait for the VM and sleep for 60 seconds for next vm to start ( boot storm if all VMs are started at once)
sleep -seconds 60
}
# send a webhook to Teams to notify that the VMs have been started
$webhookUri = ""
$body = @{
"@context" = "http://schema.org/extensions"
"@type" = "MessageCard"
"themeColor" = "d70000"
"title" = "Send Webhook to Teams"
"text" = "This is a message sent from Powershell"
}
Invoke-RestMethod -Uri $webhookUri -Method Post -Body (ConvertTo-Json -InputObject $body) 

```

> Updating a standalone Hyper-V host can be a complex and time-consuming process, but with Azure Arc and Automation Accounts, you can automate much of the work and ensure a smooth, reliable update process. Hyper-v cluster or HCI is a different story. 
{: .prompt-info }

![Desktop View](/assets/img/blog/before.png){: .normal-image}
