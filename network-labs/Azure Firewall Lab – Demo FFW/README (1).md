# Azure Firewall Lab – Demo FFW

> **Subscription:** Visual Studio Enterprise Subscription – MPN  
> **Resource Group:** demo_FFW  
> **Region:** UAE North

---

## Overview

This lab demonstrates how to deploy and configure **Azure Firewall** within a hub-and-spoke network topology to control and inspect outbound internet traffic from a private virtual machine. Traffic is routed through the firewall using a **User Defined Route (UDR)**, and access is enforced via **Firewall Policy Application Rules** — allowing permitted destinations (e.g., `www.google.com`) while blocking others (e.g., `www.facebook.com`).

---

## Architecture

```
User (RDP)
  └─► Srv-Jump (Jumpbox, Public IP)
        └─► Srv-Work (Private VM, no direct internet)
              └─► RouteTable1 → Azure Firewall (10.0.1.4)
                    └─► Firewall Policy (Application Rules)
                          ├─► ALLOW → www.google.com  ✅
                          └─► DENY  → www.facebook.com ❌
```

All communication is **zone-redundant**.

---

## Network Layout

| Resource | Subnet / Address Space | Notes |
|---|---|---|
| **VNet** | `10.0.0.0/16` | Top-level virtual network |
| **Jumpbox Subnet** | `10.0.1.0/24` | Hosts the jump server (public-facing) |
| **Workload Subnet (Workload-SN)** | `10.0.2.0/24` | Hosts the private workload VM |
| **AzureFirewallSubnet** | `10.0.1.0/26` | Dedicated subnet for Azure Firewall |
| **AzureFirewallManagementSubnet** | — | Required for forced tunneling / management traffic |

---

## Resources Deployed

### Virtual Machines

| Name | Role | Access |
|---|---|---|
| **Srv-Jump** | Jumpbox / bastion VM | Public IP, reachable via RDP from the internet |
| **Srv-Work** | Workload VM | Private only; no direct internet access |

### Azure Firewall

| Property | Value |
|---|---|
| Name | `firewall` |
| Private IP | `10.0.1.4` |
| SKU | Azure Firewall (Zone-redundant) |
| Public IP (`firewall_PIP`) | Zone-redundant |
| Management Public IP | Zone-redundant |

### Firewall Policy

| Property | Value |
|---|---|
| Name | `policeSS` |
| Type | Application Rules |
| Allowed FQDNs | `www.google.com` |
| Denied FQDNs | `www.facebook.com` (and all others not explicitly permitted) |

### Route Table

| Property | Value |
|---|---|
| Name | `RouteTable1` |
| Address Prefix | `0.0.0.0/0` (all traffic) |
| Next Hop Type | Virtual Appliance |
| Next Hop IP | `10.0.1.4` (Azure Firewall) |

### Network Security Groups (NSGs)

- Applied to **Workload-SN** – restricts lateral movement; permits internal RDP (port `3389`) from Srv-Jump only via UDR.
- Applied to **AzureFirewallManagementSubnet** – controls management plane access.

---

## Traffic Flow

### ✅ Permitted Flow (Workflow to Internet)
1. User connects via **RDP** to **Srv-Jump** (public IP).
2. Srv-Jump establishes an **internal RDP** session to **Srv-Work** (port 3389 via UDR).
3. Srv-Work attempts to reach the internet — traffic is intercepted by **RouteTable1** and forwarded to the **Azure Firewall** at `10.0.1.4`.
4. The Firewall Policy **Application Rule** evaluates the destination FQDN.
5. If the destination is `www.google.com` → traffic is **allowed** and exits via `firewall_PIP`.

### ❌ Blocked Flow (Blockflow to Website)
- If Srv-Work attempts to reach `www.facebook.com` (or any other non-whitelisted FQDN), the Firewall Policy returns a **DENY** and the connection is dropped.

---

## Prerequisites

- An active **Azure subscription** (Visual Studio Enterprise / MPN recommended for lab use)
- Permissions to create networking, compute, and firewall resources
- Azure CLI or access to the **Azure Portal**
- Basic familiarity with Azure VNet, NSG, and UDR concepts

---

## Deployment Steps

### 1. Create the Virtual Network
```bash
az network vnet create \
  --name demo_FFW-vnet \
  --resource-group demo_FFW \
  --address-prefix 10.0.0.0/16 \
  --location uaenorth
```

### 2. Create Subnets
```bash
# Jumpbox Subnet
az network vnet subnet create --vnet-name demo_FFW-vnet -g demo_FFW \
  --name JumpboxSubnet --address-prefix 10.0.1.0/24

# Workload Subnet
az network vnet subnet create --vnet-name demo_FFW-vnet -g demo_FFW \
  --name Workload-SN --address-prefix 10.0.2.0/24

# Azure Firewall Subnet (name must be exact)
az network vnet subnet create --vnet-name demo_FFW-vnet -g demo_FFW \
  --name AzureFirewallSubnet --address-prefix 10.0.1.0/26

# Azure Firewall Management Subnet (name must be exact)
az network vnet subnet create --vnet-name demo_FFW-vnet -g demo_FFW \
  --name AzureFirewallManagementSubnet --address-prefix 10.0.3.0/26
```

### 3. Deploy Public IPs
```bash
az network public-ip create -g demo_FFW --name firewall_PIP \
  --sku Standard --zone 1 2 3 --location uaenorth

az network public-ip create -g demo_FFW --name firewall_mgmt_PIP \
  --sku Standard --zone 1 2 3 --location uaenorth
```

### 4. Create Firewall Policy & Application Rules
```bash
az network firewall policy create -g demo_FFW --name policeSS --location uaenorth

az network firewall policy rule-collection-group create \
  -g demo_FFW --policy-name policeSS --name AppRuleGroup --priority 100

az network firewall policy rule-collection-group collection add-filter-collection \
  -g demo_FFW --policy-name policeSS --rule-collection-group-name AppRuleGroup \
  --name AllowGoogle --collection-priority 100 --action Allow \
  --rule-name AllowGoogle --rule-type ApplicationRule \
  --source-addresses "10.0.2.0/24" --protocols Http=80 Https=443 \
  --fqdn-tags "" --target-fqdns "www.google.com"
```

### 5. Deploy Azure Firewall
```bash
az network firewall create -g demo_FFW --name firewall \
  --location uaenorth --firewall-policy policeSS --zones 1 2 3

az network firewall ip-config create -g demo_FFW --firewall-name firewall \
  --name FW-config --public-ip-address firewall_PIP \
  --vnet-name demo_FFW-vnet
```

### 6. Create and Attach Route Table
```bash
az network route-table create -g demo_FFW --name RouteTable1 --location uaenorth

az network route-table route create -g demo_FFW --route-table-name RouteTable1 \
  --name ToFirewall --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.1.4

# Associate with Workload-SN
az network vnet subnet update -g demo_FFW --vnet-name demo_FFW-vnet \
  --name Workload-SN --route-table RouteTable1
```

### 7. Deploy Virtual Machines
- **Srv-Jump**: Deploy into `JumpboxSubnet` with a public IP. Enable RDP (port 3389) inbound.
- **Srv-Work**: Deploy into `Workload-SN` with **no public IP**. Allow internal RDP only from Srv-Jump via NSG.

---

## Validation

1. RDP into **Srv-Jump** using its public IP.
2. From Srv-Jump, RDP into **Srv-Work** using its private IP.
3. On Srv-Work, open a browser:
   - Navigate to `https://www.google.com` → **should load** ✅
   - Navigate to `https://www.facebook.com` → **should be blocked** ❌
4. Review **Azure Firewall logs** in the portal under Monitoring → Logs to confirm allow/deny entries.

---

## Key Concepts Demonstrated

| Concept | Description |
|---|---|
| **Azure Firewall** | Managed, stateful firewall-as-a-service with high availability |
| **Firewall Policy** | Centralized rule management (Application, Network, DNAT rules) |
| **UDR (User Defined Route)** | Forces all egress traffic through the firewall |
| **Zone Redundancy** | Resources span availability zones for high availability |
| **Jumpbox Pattern** | Secure access to private VMs without exposing them directly |
| **NSG** | Layer-4 traffic filtering at the subnet/NIC level |

---

## Notes

- The `AzureFirewallSubnet` name is **reserved and mandatory** — it cannot be renamed.
- The `AzureFirewallManagementSubnet` is required when using **Firewall Policy with forced tunneling**.
- Zone-redundant public IPs require **Standard SKU**.
- Azure Firewall application rules perform **FQDN-based filtering** with TLS inspection support (if enabled).

---

## License

This lab is intended for **educational and demonstration purposes** within a Microsoft Partner / Visual Studio Enterprise subscription.
