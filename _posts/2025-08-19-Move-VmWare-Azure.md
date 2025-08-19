---
title: Migrating from VMware to Azure using Azure Files
date: 2025-08-18 00:00:00 -0000
categories: [Azure]
tags: [azure]
image: "/assets/img/blog/vmwaremove/disk2vhd1.png"
---
When using a tool such as Disk2VHD, you need a storage location—and sometimes plenty of it—to export the virtual machine (VM).
But what if no local storage is available, and no USB can be connected, such as with a VMware hosting provider?


In that case, you can export the VHD directly to an Azure File Share and then copy it to Azure Blob Storage.
Note: Dynamically expanding disks are not supported in this process—you must use a fixed-size disk.


The next step is to provision a Windows Server in Azure, install Hyper-V, and use Hyper-V Manager to convert the disk to a fixed-size VHD.


After conversion, you can upload the VHD to Azure, create an image from it, and then deploy a VM.
In many cases, Azure will display an error during deployment stating that the VM cannot be customized. You can safely ignore this error.


Tools You Will Need

- Disk2VHD – for exporting the VMware VM to a VHD file.
    🔗 Download Disk2VHD
- Azure Storage Explorer – for managing Azure File Shares and Blob Storage.
    🔗 Download Azure Storage Explorer
- Hyper-V Manager – for converting VHDs from dynamic to fixed-size.
    Installed on VM in Azure


**Step 1:** Mount Azure File Share

On the source VMware server and later on the Hyper-V server in Azure, mount the Azure File Share:
Mount as Administrator because disk2vhd runs as Administrator.

![Disk2VHD Export](assets/img/blog/vmwaremove/blob.png)

**Step 2:** Export VMware VM with Disk2VHD

Run Disk2VHD on the source server.

Select the volumes you want to export.

Choose the Azure File Share as the destination path (after mounting it as a network drive).

![Disk2VHD Export](assets/img/blog/vmwaremove/disk2vhd1.png)
![Disk2VHD Uncheck](assets/img/blog/vmwaremove/uncheck.png)


**Step 3:** Convert VHD from Dynamic to Fixed

Open Hyper-V Manager on the Azure VM.

Go to Edit Disk…

Browse to the VHD file from the Azure File Share.

Select Convert → Fixed size.

![alt text](assets/img/blog/vmwaremove/fxiedsize.png)

Step 4: Copy VHD to Azure Blob Storage

You can use either Azure Storage Explorer ( Use copy and paste in the menu :) )

![alt text](assets/img/blog/vmwaremove/storageexplorer.png)

Step 5: Create Azure Managed Image & VM

In the Azure Portal, go to Images → + Create.

Select your uploaded VHD as the source.

Create an image.

Deploy a VM from that image.

![alt text](assets/img/blog/vmwaremove/azimagecreate.png)

Now you can create vm from the image.