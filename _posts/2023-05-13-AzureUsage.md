---
title: Monitor Azure usage with github actions
date: 2023-05-13 14:00:00 -0000
categories: [Azure,Usage]
tags: [azure,usage]     # TAG names should always be lowercase
---

> This will not work for Azure Plan subscriptions.
> ((400) Subscription scope usage is not supported for current api version. Please use api version after 2019-10-01)
{: .prompt-note }

### Need to keep track of your Azure usage? here is a way to use github actions to keep track of your usage. And send you a Teams message if you are over a certain amount.

> We will need to enable webhook in Teams and add the webhook to the script, and create a Service Principal with access to the subscription you want to monitor.
{: .prompt-tip }

Create Service Principal
```powershell
az ad sp create-for-rbac --name "AzureUsage" --role contributor --scopes /subscriptions/0692777c --sdk-auth
```

Github actions, it runs every day at midnight.
> We will need to add secret to the repository, AZURE_CREDENTIALS, and add the output from the command above.
{: .prompt-tip }

This is the workflow file
```yaml
# File: .github/workflows/workflow.yml

on:
  schedule:
   - cron: '0 0 * * *'  # every day at midnight

name: AzureUsage

jobs:

  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        enable-AzPSSession: true 
        
    - name: Run Azure PowerShell script
      uses: azure/powershell@v1
      with:
        inlinescript: |
          $startDate = (Get-Date).AddDays(-2).ToString("yyyy-MM-dd")
          $endDate = (Get-Date).AddDays(-1).ToString("yyyy-MM-dd")

          $twodaysago = Get-AzConsumptionUsageDetail -StartDate $startDate -EndDate $startDate | Measure-Object -Property PretaxCost -Sum | Select-Object Sum
          $yesterday = Get-AzConsumptionUsageDetail -StartDate $enddate -EndDate $enddate | Measure-Object -Property PretaxCost -Sum | Select-Object Sum
                    
          $twodaysago = [math]::Floor($twodaysago.sum)
          $yesterday = [math]::ceiling($yesterday.sum)

          $percentof = [math]::Floor($twodaysago * 1.4)

          if ($twodaysago -le $percentof -and $yesterday -lt 100) {
              Write-Host "OK: Yesterday's Azure spending ($yesterday Euro) is not 40% more than 2 days ago ($twodaysago Euro) and not more than 100 Euro | yesterday=$yesterday, spending2daysago=$twodaysago"
          }
          else
          {
          Write-Host  "Alarm!! Tom Much Azure Spending"
          $webhookUri = "webhook from teams"
          $body = @{
          "@context"   = "http://schema.org/extensions"
          "@typecod"      = "MessageCard"
          "themeColor" = "d70000"
          "title"      = "Alarm!! To Much Azure Spending"
          "text"       = "Some thing is wrong in Azure. Yesterday's Azure spending ($yesterday Euro) is 40% more than 2 days ago ($twodaysago Euro) or more than 100 Euro"
          }
          Invoke-RestMethod -Uri $webhookUri -Method Post -Body (ConvertTo-Json -InputObject $body) 
          }
        azPSVersion: "latest"
```
![Desktop View](/assets/img/blog/spending.png){: .normal-image}