# Azure Lab: VMSS with Load Balancer Inbound NAT Rules

## Overview
This lab demonstrates how to configure a **Microsoft Azure Load Balancer** to provide remote management access to individual instances within a **Virtual Machine Scale Set (VMSS)** using **Inbound NAT Rules**.

As an aspiring Cloud Engineer, this lab focuses on the mechanics of Port Translation (NAT), Network Security Group (NSG) logic, and Load Balancer Health Probes.

## Architecture
- **Virtual Machine Scale Set (VMSS):** Windows Server instances.
- **Load Balancer (Standard):** Acts as the single entry point for all traffic.
- **Inbound NAT Rules:** Maps high-range frontend ports (50000+) to the standard RDP port (3389) for specific instances.
- **Network Security Group (NSG):** Filters traffic at the subnet level based on the translated port.

## Lab Configuration Details

### 1. Load Balancer NAT Configuration
To allow RDP access, the following Inbound NAT Rule was established:
- **Type:** Backend Pool (V2)
- **Target Backend Pool:** `backendpool_WebServers`
- **Frontend Port Range Start:** `50000`
- **Backend Port:** `3389` (RDP)

**Mapping Logic:**
| Instance | Public IP | Frontend Port | Backend Port |
| :--- | :--- | :--- | :--- |
| Instance 0 | 74.162.194.79 | 50000 | 3389 |
| Instance 1 | 74.162.194.79 | 50001 | 3389 |

### 2. Network Security Group (NSG) Rules
The NSG must be configured to allow the *translated* traffic. Even though the initial connection uses port 50000, the NSG evaluates the packet after the Load Balancer has translated it back to 3389.

| Priority | Name | Port | Protocol | Source | Action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 100 | `Allow_LB_Probe` | `*` | Any | Service Tag: `AzureLoadBalancer` | Allow |
| 110 | `Allow_RDP` | `3389` | TCP | Any | Allow |

### 3. Health Probe Configuration
For the NAT rules to stay active, the Load Balancer must perceive the instances as "Healthy." In this lab, we use RDP as the heartbeat.
- **Protocol:** TCP
- **Port:** `3389`
- **Interval:** 5 seconds

## Connection Steps
1. Open **Remote Desktop Connection** (`mstsc`).
2. Enter the Public IP and assigned port: `74.162.194.79:50000`.
3. Log in using the local administrator credentials configured during deployment.

## Key Learnings
- **Port Translation:** The Load Balancer effectively hides internal management ports from the public internet by using high-range port mapping.
- **Health State Dependency:** Traffic is only routed to "Healthy" instances. If the RDP service is down or the probe is blocked, the Load Balancer will stop routing NAT traffic.
- **Scale Set Lifecycle:** When changing network settings on a Scale Set, instances may require a manual **Upgrade** via the portal to synchronize with the new Load Balancer configuration.

---
*Created as part of the Azure Labs repository.*
