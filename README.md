# Azure Cloud Engineering & Architecture Labs 🚀

Welcome to my Azure Labs repository. This project serves as a comprehensive technical portfolio where I document my journey in mastering Microsoft Azure infrastructure, security, and automation.

Each lab focuses on real-world scenarios, following the "Design-then-Deploy" methodology to ensure security, cost-efficiency, and scalability.

---

## 🏗️ Core Architectures & Methodology
In every lab within this repository, I follow these fundamental cloud principles:
- **Network-First Design:** Establishing VNet, Subnets, and NSGs before provisioning compute.
- **Least Privilege Access:** Using IP Whitelisting and RBAC to secure entry points.
- **Cost Governance:** Utilizing B-series burstable VMs and Auto-shutdown to manage resources.

---

## 📂 Lab Directory

### 🟢 Lab 1: Project Blue-Horizon (Secure Jumpbox)
* **Goal:** Deploy a hardened Windows Server 2025 management entry point.
* **Key Tech:** VNet, Subnets, NSG (/32 Whitelisting), Public IP.
* **Outcome:** A secured architecture where the server is invisible to the public internet, accessible only from a trusted administrative IP.
* [View Detailed Lab 1 Documentation](./Lab1-Project-Blue-Horizon/README.md)

### ⚪ Lab 2: Storage Architecture (Coming Soon)
* **Focus:** Managed Disks, Performance Tiers (Standard vs. Premium SSD), and Disk Lifecycle.
* **Goal:** Attach and initialize data disks to an existing workload without downtime.

### ⚪ Lab 3: Load Balancing & High Availability (Planned)
* **Focus:** Azure Load Balancer, Availability Sets, and Traffic Management.

---

## 🛠️ Tools & Technologies
- **Platform:** Microsoft Azure
- **OS:** Windows Server 2025, Linux (Ubuntu)
- **Networking:** Virtual Networks (VNet), NSGs, Azure Bastion
- **Security:** IP Whitelisting, CIDR Networking, Identity Management

---

## 📈 Learning Objectives
My goal is to demonstrate proficiency in:
1.  Designing secure network topologies.
2.  Managing cloud costs and resource optimization.
3.  Implementing hybrid-ready infrastructure.
4.  Infrastructure documentation and technical writing.

---
**Contact:** [Your Name] | [Your LinkedIn Profile]
