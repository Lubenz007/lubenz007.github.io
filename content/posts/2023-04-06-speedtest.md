---
title: Powershell Speedtest
date: 2023-04-06 20:15:00 -0000
categories: [Automation]
tags: [powershell,automation]     # TAG names should always be lowercase
summary: "PowerShell script for automated internet speed monitoring using Speedtest CLI with results stored in Azure Table Storage for analysis."
---

Monitor your internet connection performance with this PowerShell script that uses the Speedtest CLI and stores results in Azure Table Storage for historical analysis.

<!--more-->

> **Info:** The script uses the speedtest CLI from https://www.speedtest.net/apps/cli

## Problem and Solution

Was having problems with my internet connection and wanted to monitor it continuously. So I rewrote this script to use PowerShell and store the results in Azure Table Storage for tracking and analysis.

```powershell
#$StorageAccountName = "StorageAcount"
#$Key = "Storagekey"
#$StorageContext = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $Key
#$Table = (Get-AzStorageTable -Context $StorageContext | Where-Object {$_.name -eq "Speedtest"}).CloudTable
$applocation = "C:\apps\speedtest"
$path = "C:\temp\"

$SpeedTestResults=@()
$SpeedtestObj=@()

$i = 0
while ($i -eq 0)
{
    $PartitionKey = "1"
    $SpeedTestResults = & "$($applocation)\speedtest.exe" --progress=no --format=json
    $SpeedtestResults = $SpeedTestResults | ConvertFrom-Json
    $SpeedtestObj += [PSCustomObject] @{
        Time = Get-Date -Format "dd/MM/yyyy HH:mm K"
        downloadspeed = [math]::Round($SpeedtestResults.download.bandwidth / 1000000 * 8, 2)
        uploadspeed   = [math]::Round($SpeedtestResults.upload.bandwidth / 1000000 * 8, 2)
        packetloss    = ($($SpeedtestResults.packetLoss).ToString("P"))
        isp           = $SpeedtestResults.isp
        ExternalIP    = $SpeedtestResults.interface.externalIp
        InternalIP    = $SpeedtestResults.interface.internalIp
        UsedServer    = $SpeedtestResults.server.host
        location      = $SpeedTestResults.server.location
        Jitter        = [math]::Round($SpeedtestResults.ping.jitter)
        Latency       = [math]::Round($SpeedtestResults.ping.latency)       
    }
    # ---- Move to table storage ----
    # Add-AzTableRow -table $Table -PartitionKey $PartitionKey -RowKey (Get-Date).Ticks -property $SpeedtestObj
   
    Start-Sleep -Seconds 15
}
#$SpeedtestObj | Format-Table | Out-String|ForEach-Object {Write-Host $_}
$SpeedtestObj | Export-Csv -Path $path\speedtest.csv -NoTypeInformation
```

