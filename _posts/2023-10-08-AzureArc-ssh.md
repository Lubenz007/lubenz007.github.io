---
title: Azure Arc - (Azure Cli - ssh - rdp)
date: 2023-10-08 02:00:00 -0000
categories: [Azure]
tags: [azure]     # TAG names should always be lowercase
---
If you have installed Azure Arc.
You can use Windows Admin Center for Windows Machines, but you can also enabled OpenSSH Extesion on Windows Server and allow SSH to tunnel to local host or other machines that allow RDP/SSH.

So how do we do this.
First you need to install Azure CLi on your local computer if you dont have that installed.
Open powershell and "winget install -e --id Microsoft.AzureCLI"
Or if you have any other OS.
[How to install the Azure CLI | Microsoft Learn](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

Enable OpenSSH exstension on the VM that has Azure Arc installed.

Logon to Azure With Azure CLi
PS C:\> az login
Ps.
Please use MFA.

The we go to the Connect settings and select password authentication on the VM

Copy the connect string and paste to Powershell and add "-L 3333:192.168.x.x:3389" the ip of the host you want to connect to.
az ssh arc --subscription "asdfasdfadf-adfadsfadsf-sdfc" --resource-group "Arc-Servers" --name "HYPER-01" --local-user "administrator" "-L 3333:192.168.81.25:3389"


And you are connect over SSH tunnel with RDP.
