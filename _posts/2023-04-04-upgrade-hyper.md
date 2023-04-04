---
title: Upgrade Hyper-V Hosts with Azure Arc and Automation Accounts
date: 2023-04-04 22:15:00 -0000
categories: [Hyper-V,Azure,Automation]
tags: [hyperv,azure,automation]     # TAG names should always be lowercase
---

Azure Arc is a powerful tool for managing and updating hybrid infrastructure, including standalone Hyper-V hosts. In this blog post, we will walk you through how to use Azure Arc to update a standalone Hyper-V host, using a scheduled update task in Automation Account and pre and post scripts that shut down and start up VMs.

Before we begin, it's worth noting that Hyper-V host updates can be complex and require careful planning to avoid disruption to your environment. Always make sure you have a comprehensive backup plan in place, and test your update procedures thoroughly in a non-production environment before applying them to your production systems.

With that said, let's get started!

Step 1: Set up Azure Arc

The first step is to set up Azure Arc for your standalone Hyper-V host. This involves deploying the Azure Arc agent to the host and registering it with your Azure subscription. You can find detailed instructions for this process in the Azure documentation.

Step 2: Create an Automation Account

Next, you'll need to create an Automation Account in your Azure subscription. This account will be used to manage the update process for your Hyper-V host. You can create an Automation Account from the Azure portal by navigating to the Automation Accounts blade and clicking Add.

Step 3: Create a Scheduled Update Task

Once you have an Automation Account set up, you can create a Scheduled Update Task to automate the update process for your Hyper-V host. To do this, navigate to the Update Management blade in your Automation Account, and click Schedule Update Deployment. From here, you can select the target Hyper-V host, set a schedule for the update task, and configure any additional options you need.

Step 4: Create Pre and Post Scripts

To ensure a smooth update process for your Hyper-V host, you'll need to create pre and post scripts that shut down and start up VMs before and after the update. These scripts will be run automatically as part of the update task, and can be customized to suit your specific needs.

For example, your pre-script might look something like this:

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
```
This script will find all running VMs on the Hyper-V host and shut them down before the update is applied. Save the names of the running VMs to a file so that they can be started up again after the update is complete.

Your post-script might look like this:

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
```

This script will find all VMs that were shut down during the update process and start them up again.

Step 5: Run the Update Task

With your Automation Account, Scheduled Update Task, and pre and post scripts in place, you're ready to run the update task. Simply navigate to the Update Management blade in your Automation Account, select the target Hyper-V host, and click Schedule Update Deployment.

The update process will then begin, with the pre-script shutting down VMs and the update being applied to the Hyper-V host. Once the update is complete, the post-script will start up any VMs that were shut down, and your Hyper-V host will be up-to-date and ready to go.

Conclusion

Updating a standalone Hyper-V host can be a complex and time-consuming process, but with Azure Arc and Automation Accounts, you can automate much of the work and ensure a smooth, reliable update process. By following the steps outlined in this blog post, you can easily set up a Scheduled Update Task with pre and post scripts to manage your Hyper-V host updates with ease.

![Desktop View](/assets/img/blog/before.png){: .normal-image}
![Desktop View](/assets/img/blog/runbook.png){: .normal-image}
![Desktop View](/assets/img/blog/scheduletask.png){: .normal-image}
![Desktop View](/assets/img/blog/prepost.png){: .normal-image}