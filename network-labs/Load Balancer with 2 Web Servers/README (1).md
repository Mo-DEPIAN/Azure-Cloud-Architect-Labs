# Azure Lab 1 — Load Balancer with 2 Web Servers

## Overview

This lab demonstrates how to configure an **Azure Load Balancer** that distributes incoming HTTP/HTTPS traffic across two Windows or Linux web server virtual machines (VMs) inside an Azure Virtual Network.

---

## Architecture

```
Internet / Client
       |
  Public IP Address (Frontend)
       |
  Azure Load Balancer (Standard SKU)
  - Rule: Round-robin on port 80
  - Health probe: TCP port 80
       |
  ┌────┴────┐
  │         │
VM 1       VM 2
Web Server  Web Server
10.0.1.4   10.0.1.5
  └────┬────┘
  BackendSubnet (10.0.1.0/24)
  VNet-Lab1 (10.0.0.0/16)
```

---

## Azure Resources

| Resource | Name | Details |
|---|---|---|
| Resource Group | RG-Lab1 | Contains all lab resources |
| Virtual Network | VNet-Lab1 | Address space: 10.0.0.0/16 |
| Subnet | BackendSubnet | Address range: 10.0.1.0/24 |
| Public IP | pip-lab1 | Standard SKU, static |
| Load Balancer | lb-lab1 | Standard SKU, round-robin |
| Virtual Machine 1 | vm-web-01 | Private IP: 10.0.1.4 |
| Virtual Machine 2 | vm-web-02 | Private IP: 10.0.1.5 |

---

## Lab Objectives

- Create a Virtual Network and subnet in Azure
- Deploy two web server VMs in the backend subnet
- Configure an Azure Standard Load Balancer with a public IP
- Set up a load balancing rule on port 80 (HTTP)
- Configure a health probe to monitor VM availability
- Verify traffic is distributed across both VMs

---

## Prerequisites

- Active Azure subscription
- Access to the Azure Portal or Azure CLI
- Basic knowledge of Azure networking and virtual machines

---

## Steps

### 1. Create the Resource Group
```bash
az group create --name RG-Lab1 --location eastus
```

### 2. Create the Virtual Network and Subnet
```bash
az network vnet create \
  --resource-group RG-Lab1 \
  --name VNet-Lab1 \
  --address-prefix 10.0.0.0/16 \
  --subnet-name BackendSubnet \
  --subnet-prefix 10.0.1.0/24
```

### 3. Create the Public IP Address
```bash
az network public-ip create \
  --resource-group RG-Lab1 \
  --name pip-lab1 \
  --sku Standard \
  --allocation-method Static
```

### 4. Create the Load Balancer
```bash
az network lb create \
  --resource-group RG-Lab1 \
  --name lb-lab1 \
  --sku Standard \
  --public-ip-address pip-lab1 \
  --frontend-ip-name FrontendConfig \
  --backend-pool-name BackendPool
```

### 5. Add Health Probe
```bash
az network lb probe create \
  --resource-group RG-Lab1 \
  --lb-name lb-lab1 \
  --name HealthProbe \
  --protocol tcp \
  --port 80
```

### 6. Add Load Balancing Rule
```bash
az network lb rule create \
  --resource-group RG-Lab1 \
  --lb-name lb-lab1 \
  --name HTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name FrontendConfig \
  --backend-pool-name BackendPool \
  --probe-name HealthProbe
```

### 7. Deploy Web Server VMs
Deploy `vm-web-01` and `vm-web-02` into `BackendSubnet`, then install and start a web server (IIS or Apache/Nginx) on each.

### 8. Add VMs to Backend Pool
Attach each VM's NIC to the load balancer backend pool.

### 9. Test
Open a browser and navigate to the public IP address. Refresh multiple times to confirm traffic is being distributed between both VMs.

---

## Expected Result

Both web servers respond to HTTP requests. The load balancer distributes traffic evenly using round-robin. If one VM goes down, the health probe detects it and routes all traffic to the healthy VM.

---

## Notes

- Use **Standard SKU** for both the Public IP and Load Balancer (Basic SKU does not support availability zones).
- Ensure the **Network Security Group (NSG)** allows inbound traffic on port 80.
- Health probes require the web service to be running and responding on the configured port.

---

## Author

Azure Lab — Load Balancer Series
