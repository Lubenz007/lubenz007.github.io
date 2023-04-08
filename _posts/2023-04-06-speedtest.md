---
title: Powershell Speedtest
date: 2023-04-06 20:15:00 -0000
categories: [Powershell,Automation]
tags: [powershell,automation]     # TAG names should always be lowercase
---
## Was having problems with my internet connection and wanted to monitor it. So I rewrote this script to use powershell and store the results in a table storage.

> The script is using the speedtest cli from https://www.speedtest.net/apps/cli
{: .prompt-info }

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

<table>
  {% for row in site.data.speedtest %}
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
