# Azure Disk Encryption with Customer-Managed Keys (CMK)

## Project Overview
This lab demonstrates the implementation of **Azure Disk Encryption (ADE)** using a **Disk Encryption Set (DES)** and **Azure Key Vault**. By moving away from platform-managed keys, this project showcases a "Security-First" approach to infrastructure, ensuring that virtual machine disks are encrypted with keys fully controlled by the administrator.

## Technical Architecture
The deployment follows a secure workflow to establish a chain of trust between the compute resources and the cryptographic keys:
1.  **Azure Key Vault:** Acts as the hardened repository for the encryption keys.
2.  **Encryption Key:** A dedicated RSA key generated within the vault.
3.  **Disk Encryption Set (DES):** Acts as a bridge between the Virtual Machine and the Key Vault.
4.  **Managed Identity:** The DES is granted specific permissions (Get, Wrap, and Unwrap) via Key Vault Access Policies to retrieve and use the key.

## Lab Objectives
- Provision a Key Vault with **Bypass/Purge Protection** enabled.
- Generate a cryptographic key for disk encryption.
- Create and configure a **Disk Encryption Set**.
- Assign **Role-Based Access Control (RBAC)** or Access Policies for the DES Identity.
- Deploy/Update a Virtual Machine to utilize the DES for OS and Data disks.

## Prerequisites
- An active Azure Subscription.
- Azure CLI or PowerShell (optional for verification).
- Permissions to create Resource Groups and manage Key Vaults.

## Implementation Steps

### 1. Key Vault Configuration
The Key Vault must be configured to support disk encryption. This includes enabling **Purge Protection** and **Soft Delete** to prevent accidental loss of encryption keys, which would render the disks unreadable.

### 2. Key Generation
A new key was generated within the vault with the following specifications:
- **Key Type:** RSA
- **RSA Key Size:** 2048 (standard for ADE)

### 3. Disk Encryption Set (DES) Setup
The DES was created in the same region as the resources to be encrypted. Upon creation, the DES generates a **System-Assigned Managed Identity**.

### 4. Permissions & Access Control
To allow the DES to encrypt and decrypt disks, the following permissions were granted in the Key Vault Access Policies:
- **Key Permissions:** `Get`, `Wrap Key`, `Unwrap Key`.
- **Principal:** The Managed Identity of the Disk Encryption Set.

### 5. VM Encryption
The Virtual Machine was configured to use the Disk Encryption Set. This ensures that any data written to the VHDs is encrypted at rest using the key stored in the Key Vault.

## Verification
To verify the encryption status, use the following Azure CLI command:
```bash
az vm encryption show --name <VM-Name> --resource-group <RG-Name>
