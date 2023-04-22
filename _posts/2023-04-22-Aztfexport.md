---
title: Getting started with Terraform and Azure
date: 2023-04-22 18:46:49:00 -0000
categories: [Azure,Terraform]
tags: [azure,terraform]     # TAG names should always be lowercase
---

I like to watch videos on how to do things, but i don't like to start from scratch. I like to have a template from which i can start and test things out. So start to look out for exporting Azure config to templates. Terraformer is a tool that can export existing cloud infrastructure to Terraform code. But I like aztfexport better because it is more Azure-specific.

## Install aztfexport is easy from package manager
```powershell
c:\winget install aztfexport
```
## And we need also Azure Cli
```powershell
c:\winget install azure-cli
```
## Login to Azure
This was the part I struggled with: where is the config file? It was too simple: login to Azure with Azure Cli and then select the subscription I wanted to export from. Terraform, Terraformer, and aztfexport will use the same subscription as Azure Cli.
> So if you try to export and it fails to run, you need to login to Azure Cli.
{: .prompt-tip }

```powershell
c:\az login
```
## Select subscription
```powershell
c:\az account list -o table
c:\az account set --subscription "MySubscription"
```
## Export to Terraform
So I start with a simple resource group.
And export it to Terraform's current folder.
```powershell
c:\dashboard\aztfexport rg Dashboard
```
There will be a menu list, and we will use w to import Terraform files.

![Desktop View](/assets/img/blog/terra1.png){: .normal-image}

## Result is a Terraform files
So we can start from there and try out things.
![Desktop View](/assets/img/blog/terra2.png){: .normal-image}

The first is to test if the Terraform plan works.
```powershell
c:\dashboard\terraform plan
```
![Desktop View](/assets/img/blog/terra3.png){: .normal-image}
Bingo, the plan works, so we can start to add more resources to the Terraform files.







