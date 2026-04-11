# Project Blue-Horizon: Secure Azure Management Jumpbox

## 📌 Project Overview
This repository contains the architectural design and deployment details for **Project Blue-Horizon**, a secure cloud entry point (Jumpbox) built on Microsoft Azure. The project demonstrates a "Network-First" architecture, ensuring that compute resources are deployed into a pre-configured, hardened environment.

## 🏗️ Architecture Design
The infrastructure follows a strict dependency-based sequence:

* **Resource Group (`rg-bluehorizon-prod-001`):** A logical container used for grouping all lab assets for easier management and billing.
* **Virtual Network (`vnet-bluehorizon-001`):** The private network "land" (10.0.0.0/16) that isolates the environment from the public internet.
* **Subnet (`snet-management`):** A dedicated "yard" (10.0.1.0/24) specifically for management and administrative tools.
* **Network Security Group (`nsg-management-001`):** The "Gatekeeper" controlling traffic. It uses **IP Whitelisting** to restrict RDP (Port 3389) access to a single authorized Public IP address.
* **Compute (`vm-jumpbox-001`):** A Windows Server 2025 Virtual Machine (B2s size) acting as the management console.
* **Storage:** A 128 GB Premium SSD to ensure high-performance OS operations.

## 🛠️ Configuration Details
- **Location:** [Insert your Azure Region here, e.g., East US]
- **Public IP:** Static/Dynamic assigned to the VM (protected by NSG).
- **Governance:** **Auto-Shutdown** is enabled for 19:00 (Local Time) to preserve Azure credits.

## 🛡️ Security Strategy: The "Whitelisting" Method
This lab focuses on securing the management entry point. Instead of opening Port 3389 to the world, the NSG rule uses a **/32 CIDR** notation.
* **Logic:** Azure only accepts traffic from the administrator's specific Public IP (Router address). 
* **Result:** All "Brute Force" RDP attacks from unauthorized IPs are dropped at the network edge.

## 📊 Key Learnings
- **Private vs. Public IP:** Understanding how local private IPs (192.168.x.x) map to public router IPs in the cloud.
- **Subnetting:** Designing network boundaries before deploying servers.
- **Cost Management:** Practicing "Burstable" VM sizes and automated shutdown schedules to manage cloud budgets.

---
*Developed as part of the Azure Cloud Architect Training Path.*
