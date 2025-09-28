---
title: Azure Arc - (Azure Cli - ssh - rdp)
date: 2023-10-08 02:00:00 -0000
categories: [Azure]
tags: [azure]     # TAG names should always be lowercase
summary: "Learn how to use Azure Arc with Azure CLI to establish SSH connections and RDP tunnels to Windows machines through OpenSSH extension."
---
If you have deployed Azure Arc, you have the option to utilize Windows Admin Center for Windows Machines. Additionally, you can enable the OpenSSH Extension on Windows Server, granting the capability to establish SSH tunnels to the localhost or other machines that support RDP/SSH connections.

<!--more-->
![Desktop View](/assets/img/blog/SSH-Tunnel.png)

To begin the process, please follow these steps:

Ensure that you have the Azure CLI installed on your local computer. If it is not already installed, open PowerShell and execute the command: winget install -e --id Microsoft.AzureCLI.

If you are using a different operating system, refer to the official guide on how to install the Azure CLI provided by Microsoft Learn: 
[How to install the Azure CLI | Microsoft Learn.](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

Next, enable the OpenSSH extension on the VM where Azure Arc is deployed.

![Desktop View](/assets/img/blog/Enable-SSH.png)

![Desktop View](/assets/img/blog/Settings-Extensions.png)

- Logon to Azure With Azure CLi

```powershell
PS C:\> az login
```

> **Tip:** Please use Multi-Factor Authentication for added security.

The we go to the Connect settings and select password authentication on the VM
![Desktop View](/assets/img/blog/Connect_string.png)

Copy the connect string and paste to Powershell and add "-L 3333:192.168.x.x:3389" the ip of the host you want to connect to.

```powershell
az ssh arc --subscription "asdfasdfadf-adfadsfadsf-sdfc" --resource-group "Arc-Servers" --name "HYPER-01" --local-user "administrator" "-L 3333:192.168.81.25:3389"
```

![Desktop View](/assets/img/blog/RDP_Session1.png)

![Desktop View](/assets/img/blog/RDP_Session2.png)

And you are connect over SSH tunnel with RDP.
