---
title: Secure Remote Management with Windows Admin Center via Azure Arc and SSH Tunneling.
date: 2025-12-17 00:00:00 +0000
categories: [Azure]
tags: [azure]
image: /assets/img/blog/update-1200x630.png
---

## Overview

This guide demonstrates how to securely access Windows Admin Center (WAC) on a remote server using Azure Arc and SSH tunneling. This approach provides a secure, cloud-based connection without exposing your management interface directly to the internet.

### What You'll Achieve

- Secure remote access to Windows Admin Center through Azure Arc
- SSH tunnel for encrypted port forwarding
- Centralized management of multiple servers from a single WAC instance
- Leverage WAC extensions (Active Directory, DNS, RDP, etc.) remotely

---

## Prerequisites

Before you begin, ensure you have:

- **Azure subscription** with appropriate permissions
- **Azure CLI** installed on your local machine
- **Windows Admin Center** installed on the target server
- **Azure Arc agent** installed on the server hosting Windows Admin Center
- **Local administrator credentials** for the Arc-enabled server

---

## Architecture

```
[Your Local Machine] 
    ↓ (Azure CLI + SSH)
[Azure Arc Service]
    ↓ (Secure Connection)
[Arc-enabled Server with WAC]
```