---
title: hyper-v-tool 
date: 2023-09-30 14:00:00 -0000
categories: [Hyper-v]
tags: [hyperv ]     # TAG names should always be lowercase
image: "https://github.com/Lubenz007/hyper-v-tool/assets/116028026/1e961bfd-aa70-41c3-96dd-6740f175d03b"
summary: "Custom PowerShell tool for creating VMs in Hyper-V lab environment using golden images with automated SSH-based deployment and menu system."
---

My tool to create VMs in my lab environment with streamlined automation.

**GitHub Repository:** https://github.com/Lubenz007/hyper-v-tool

<!--more-->

Utilizing a standalone Hyper-V server and generating VMs from a golden image might be considered an older approach, but it's one I personally prefer. Over the years, I've diligently maintained a tool for crafting VMs on my Hyper-V host within the lab environment. I employ SSH to connect to the Hyper-V host and initiate the VM creation process. This tool provides a streamlined menu system that automatically configures the VM, allowing me to promptly begin using it.
* All config is done in the vm_menu.ps1
 
-  You can find the oscdimg.exe in the Windows 11 22h2 ADK: 
-  https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install 
-  Kit "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools\arm64\Oscdimg"
 
- $global:oscdimgPath = "$global:StartupFolder\tools\oscdimg.exe"

- #Template location
- $global:template = "$global:StartupFolder\template"

- #template vhd name / windows is sysprep / Ubuntu cloud-init / modify for your template.
- $global:2022core = "W2022C.vhdx"
- $global:2022stand = "W2022S_OS.vhdx"
- $global:2022data = "W2022D_OS.vhdx"
- $global:Ubuntu = "Ubuntu_OS.vhd"

![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/1e961bfd-aa70-41c3-96dd-6740f175d03b)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/24f16f37-738a-4a17-a990-238896e9bcb3)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/ac87298b-dd9d-4d1c-8d06-3db92c6105bf)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/817222df-cee3-40ee-8a39-740abea0b3d5)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/4e5d853f-7b6e-47ef-9827-2b0f67573578)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/9ec272c6-86e3-4c96-8b13-41ae1f15b132)
![image](https://github.com/Lubenz007/hyper-v-tool/assets/116028026/b69fe441-64d9-4d7a-85b6-c7e80df0796a)