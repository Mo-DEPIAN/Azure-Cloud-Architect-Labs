# Project Blue-Horizon: Stealth-Admin & Secure Gateway Lab
## Implementation of Azure Bastion for Zero-Exposure Management

### 🛠 Overview
This lab demonstrates the implementation of a **"Zero-Exposure"** management loop. By utilizing **Azure Bastion**, I established a secure, browser-based RDP/SSH entry point to a virtual machine that has **no Public IP address**. This architecture effectively eliminates the common attack vectors associated with open RDP/SSH ports (3389/22) on the public internet.

**Identifier:** Depian | **Project:** Blue-Horizon | **Focus:** Cloud Security & Infrastructure

---

### 🏗 Architecture Summary
The lab follows a security-hardened flow where management traffic never leaves the Azure private backbone once it enters the gateway:

1.  **Public Request:** Connection via Browser over **HTTPS (Port 443)**.
2.  **Proxy Translation:** Azure Bastion service "unwraps" the request in the `AzureBastionSubnet`.
3.  **Private Final Hop:** Traffic is forwarded to the VM via **RDP (3389)** using the internal private network.

---

### 🔒 Network & Security Configuration

#### 1. IP Addressing (CIDR)
* **VNet Address Space:** `172.16.0.0/16`
* **AzureBastionSubnet:** `172.16.0.0/26` (Range: .0 - .63)
* **WorkloadSubnet:** `172.16.1.0/24`

#### 2. NSG "Least Privilege" Rules
To ensure the **"Locked Vault"** state, the following Inbound rules were applied to the VM's subnet:

| Priority | Name | Port | Protocol | Source | Action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 100 | AllowBastionInbound | 3389 | TCP | 172.16.0.0/26 | **Allow** |
| 65001 | DenyAllInbound | Any | Any | Any | **Deny** |

*By overriding default rules, I ensured that the VM only responds to the Bastion gateway.*

---

### 🚀 Implementation Steps
1.  **Network Setup:** Provisioned `VNet-Secure` with segregated subnets.
2.  **Isolated VM:** Deployed a target VM with **Public IP: None** and **Inbound Ports: None**.
3.  **Gateway Provisioning:** Deployed Azure Bastion into the dedicated subnet.
4.  **Security Hardening:** Configured NSG rules to restrict source traffic to the Bastion IP range.
5.  **Validation:** Verified the secure tunnel via HTML5 browser rendering.

---

### 📂 Lab Documentation
*Screenshots for this lab are organized in the `/images` folder, following the `01_step_name.png` format to preserve deployment order.*

---
*Developed by **Depian** | MSCS Portfolio - Project Blue-Horizon*
