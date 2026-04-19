# Azure Lab: VNet Peering & Security Bypass for Connectivity

## 📌 Project Overview
This lab demonstrates the establishment of a private network bridge between two isolated Azure Virtual Networks (VNets). The goal was to achieve successful ICMP (Ping) communication between Virtual Machines located in different VNets by configuring both Azure infrastructure and the Windows guest OS.

## 🛠️ Technologies Used
* **Cloud Provider:** Microsoft Azure
* **Networking:** Virtual Network (VNet), VNet Peering
* **Compute:** Azure Virtual Machines (Windows Server)
* **Security:** Network Security Groups (NSG), Windows Defender Firewall
* **Automation:** PowerShell

## 🏗️ Architecture
1. **Source VNet:** `VM02-vnet` (IP Space: 10.0.0.0/16)
2. **Target VNet:** `RG1_Vnet1`
3. **Connectivity:** Global VNet Peering (Low-latency Microsoft Backbone)

## 🚀 Lab Steps

### 1. VNet Peering Setup
Connected `VM02-vnet` to `RG1_Vnet1`.
* **Configuration:** Enabled "Allow forwarded traffic" and "Allow access to remote VNet."
* **Status:** Verified as **Connected**.

### 2. Infrastructure Security (NSG)
Updated the Inbound Security Rules in the Network Security Group:
* **Protocol:** ICMP
* **Action:** Allow
* **Description:** Allowed ICMP traffic from the peered VNet range to verify connectivity.

### 3. OS Security Bypass (PowerShell)
To ensure the Windows OS did not drop packets, I used PowerShell to disable the guest OS firewall across all profiles.

**Command executed:**
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
