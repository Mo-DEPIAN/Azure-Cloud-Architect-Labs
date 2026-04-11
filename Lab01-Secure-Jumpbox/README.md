# Lab 1: Project Blue-Horizon — Secure Private Cloud Foundation

## 📝 Lab Scenario
A business requires a secure, private cloud environment to host administrative tools. The primary requirement is to ensure that management ports (RDP) are never exposed to the public internet, protecting the infrastructure from brute-force attacks and unauthorized access.

## 🏗️ Architecture Design
This lab follows a **Network-First** approach. I established the security perimeter before deploying any compute resources.

### 1. Networking (The Foundation)
* **Virtual Network (VNet):** Created `vnet-bluehorizon-001` with a `10.0.0.0/16` address space.
* **Management Subnet:** Defined a specific subnet (`10.0.1.0/24`) to segment management traffic from future application workloads.

### 2. Security (The Gatekeeper)
* **Network Security Group (NSG):** Implemented `nsg-management-001`.
* **IP Whitelisting:** Configured an Inbound Security Rule for **Port 3389 (RDP)** using a **/32 CIDR mask**. This ensures that ONLY my specific public router IP can knock on the door of the VM.
* **Default Deny:** All other inbound traffic from the internet is dropped by default.

### 3. Compute & Governance
* **Virtual Machine:** Deployed a **Windows Server 2025** instance (B2s burstable size).
* **Cost Management:** Enabled **Auto-Shutdown** for 19:00 daily to optimize Azure credit consumption.

---

## 🖼️ Lab Evidence

### Network Security Group Configuration
Below is the configuration showing the restricted source IP rule that secures the environment.
![NSG Rules](./screenshots/nsg-rules.png)

### Successful Remote Connection
Proof of secure RDP connectivity to the Windows Server 2025 environment.
![RDP Success](./screenshots/rdp-success.png)

---

## 📊 Key Learnings
- **Private vs. Public Addressing:** Confirmed that Azure sees the "Street Address" (Public IP) of my router
