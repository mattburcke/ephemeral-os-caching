# Ephemeral OS Disk VM Deployment with Azure Bastion Developer

This template deploys an Azure Virtual Machine with an ephemeral OS disk and Azure Bastion Developer SKU for secure remote access without public IP exposure.

Available in both ARM Template (JSON) and Bicep formats.

## Files Included

- **vm-ephemeral.json** - ARM Template (JSON format)
- **vm-ephemeral.parameters.json** - ARM Template parameters file
- **vm-ephemeral.bicep** - Bicep template
- **vm-ephemeral.bicepparam** - Bicep parameters file

## What is an Ephemeral OS Disk?

Ephemeral OS disks are stored on the local VM storage rather than on Azure Storage. Benefits include:

- **Lower cost**: No storage costs for the OS disk
- **Faster provisioning**: Quicker VM creation and reimaging
- **Better performance**: Lower read/write latency
- **Automatic cleanup**: Disk is deleted when VM is deallocated or deleted

## Prerequisites

- Azure subscription
- Azure CLI installed (with Bastion extension for SSH/RDP tunneling)
- Appropriate VM size that supports ephemeral OS disks

## Network Architecture

The deployment creates:
- **Virtual Network** (10.0.0.0/16)
  - Default subnet (10.0.0.0/24) for the VM
  - AzureBastionSubnet (10.0.1.0/26) for Bastion
- **Azure Bastion Developer SKU** - Secure RDP/SSH access without public IPs
- **Network Security Group** - NSG attached to VM subnet (no RDP rules needed)
- **No public IP on VM** - Enhanced security posture

## Supported VM Sizes

Not all VM sizes support ephemeral OS disks. Ensure your chosen size has sufficient cache or temp disk size. The template restricts to known compatible sizes like:
- Standard_DS2_v2, Standard_DS3_v2
- Standard_D2s_v3, Standard_D4s_v3
- Standard_E2s_v3, Standard_E4s_v3

## Key Configuration

The template configures the ephemeral disk with:

**Bicep:**
```bicep
osDisk: {
  caching: 'ReadWrite'  // Full caching for best performance
  diffDiskSettings: {
    option: 'Local'
    placement: 'CacheDisk'  // or 'ResourceDisk'
  }
}
```

**ARM Template:**
```json
"osDisk": {
  "caching": "[parameters('diskCaching')]",
  "diffDiskSettings": {
    "option": "Local",
    "placement": "[parameters('diffDiskPlacement')]"
  }
}
```

### Placement Options

- **CacheDisk**: Uses the VM's cache disk (requires sufficient cache space)
- **ResourceDisk**: Uses the VM's temporary disk (requires sufficient temp disk space)

## Deployment

### Option 1: ARM Template with Azure CLI

```bash
# Create a resource group
az group create --name rg-ephemeral-vm --location eastus

# Deploy the ARM template
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.json \
  --parameters vm-ephemeral.parameters.json \
  --parameters adminPassword='YourSecurePassword123!'

# Wait for deployment to complete (Bastion can take 5-10 minutes)
```

### Option 2: ARM Template with PowerShell

```powershell
# Create a resource group
New-AzResourceGroup -Name rg-ephemeral-vm -Location eastus

# Deploy the ARM template
New-AzResourceGroupDeployment `
  -ResourceGroupName rg-ephemeral-vm `
  -TemplateFile ./vm-ephemeral.json `
  -TemplateParameterFile ./vm-ephemeral.parameters.json `
  -adminPassword (ConvertTo-SecureString 'YourSecurePassword123!' -AsPlainText -Force)
```

### Option 3: Bicep with Azure CLI

```bash
# Create a resource group
az group create --name rg-ephemeral-vm --location eastus

# Deploy the Bicep template
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.bicep \
  --parameters vm-ephemeral.bicepparam \
  --parameters adminPassword='YourSecurePassword123!'

# Wait for deployment to complete (Bastion can take 5-10 minutes)
```

### Option 4: Inline parameters (ARM or Bicep)

```bash
# ARM Template
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.json \
  --parameters vmName='myvm' \
               adminUsername='azureadmin' \
               adminPassword='YourSecurePassword123!' \
               vmSize='Standard_DS2_v2' \
               location='eastus'

# Bicep Template
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.bicep \
  --parameters vmName='myvm' \
               adminUsername='azureadmin' \
               adminPassword='YourSecurePassword123!' \
               vmSize='Standard_DS2_v2' \
               location='eastus'
```

## Important Considerations

1. **Secure Access**: VM has no public IP. Access is through Azure Bastion Developer SKU only.

2. **Bastion Developer SKU**: 
   - Lower cost option for development scenarios
   - Supports single VM connection at a time
   - Uses tunneling via Azure CLI
   - Ideal for dev/test environments

3. **Data Persistence**: Ephemeral OS disks are reset when the VM is:
   - Stopped/Deallocated
   - Reimaged
   - Deleted
   
   Store important data on data disks or external storage.

4. **VM Size Requirements**: The VM must have sufficient cache or temp disk space for the OS disk.

5. **Caching**: Must be set to `ReadOnly` for ephemeral OS disks.

6. **No Disk Backup**: Cannot use Azure Backup directly on ephemeral OS disks.

7. **Best Use Cases**:
   - Stateless applications
   - Development/test environments
   - Containerized workloads
   - Scale sets with golden images

## Connecting to the VM

### Using Azure CLI (Recommended)

The deployment outputs provide a ready-to-use connect command:

```bash
# Get the connect command from outputs
az deployment group show \
  --resource-group rg-ephemeral-vm \
  --name <deployment-name> \
  --query properties.outputs.connectCommand.value -o tsv

# Or connect directly using:
az network bastion ssh \
  --name <bastion-name> \
  --resource-group rg-ephemeral-vm \
  --target-resource-id <vm-resource-id> \
  --auth-type password \
  --username azureadmin

# For RDP (Windows):
az network bastion rdp \
  --name <bastion-name> \
  --resource-group rg-ephemeral-vm \
  --target-resource-id <vm-resource-id>
```

### Using Azure Portal

1. Navigate to the VM in Azure Portal
2. Click "Connect" > "Bastion"
3. Enter username and password
4. Click "Connect"

## Outputs

The template provides the following outputs:
- `adminUsername`: The admin username for the VM
- `privateIPAddress`: The private IP address of the VM
- `vmId`: The resource ID of the VM
- `bastionName`: The name of the Bastion host
- `connectCommand`: Ready-to-use Azure CLI command to connect via Bastion

## Clean Up

```bash
az group delete --name rg-ephemeral-vm --yes --no-wait
```

## Security Recommendations

- ✅ No public IP on VM (implemented)
- ✅ Azure Bastion for secure access (implemented)
- Use Azure Key Vault to store passwords
- Implement Just-In-Time VM access policies
- Enable Azure Security Center recommendations
- Consider using certificate-based authentication instead of passwords
