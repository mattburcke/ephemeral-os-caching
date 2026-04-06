# Azure VM with Ephemeral OS Disk and Bastion Developer

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FYOUR-USERNAME%2FYOUR-REPO%2Fmain%2Fvm-ephemeral.json)

Deploy a Windows Server 2022 virtual machine with an ephemeral OS disk featuring **full caching** (Public Preview) and Azure Bastion Developer SKU for secure, cost-optimized remote access.

> 📢 **Latest:** This template leverages the new [Ephemeral OS Disk with Full Caching](https://techcommunity.microsoft.com/blog/AzureCompute/public-preview-ephemeral-os-disk-with-full-caching-for-vmvmss/4500191) feature announced March 30, 2026, providing **10X better IO performance** and sub-millisecond latency.

## Features

✅ **Ephemeral OS Disk with Full Caching** - Entire OS disk cached on local storage (Public Preview)  
✅ **10X Better IO Performance** - Sub-millisecond latency for OS disk operations  
✅ **Enhanced Resilience** - Zero dependency on remote storage for OS disk reads  
✅ **Asynchronous Caching** - No impact to VM creation times  
✅ **Azure Bastion Standard SKU** - Secure RDP/SSH access without public IP exposure  
✅ **No Public IP on VM** - Enhanced security posture  
✅ **Flexible Placement** - Choose between CacheDisk or ResourceDisk for ephemeral storage  
✅ **ARM Template & Bicep** - Available in both formats

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│ Resource Group                                          │
│                                                         │
│  ┌──────────────────────────────────────────┐          │
│  │ Virtual Network (10.0.0.0/16)            │          │
│  │                                          │          │
│  │  ┌─────────────────────────────────┐    │          │
│  │  │ Default Subnet (10.0.0.0/24)    │    │          │
│  │  │                                 │    │          │
│  │  │  ┌──────────────────────────┐   │    │          │
│  │  │  │ VM (Ephemeral OS Disk)   │   │    │          │
│  │  │  │ - No Public IP           │   │    │          │
│  │  │  │ - Private IP Only        │   │    │          │
│  │  │  └──────────────────────────┘   │    │          │
│  │  └─────────────────────────────────┘    │          │
│  │                                          │          │
│  │  ┌─────────────────────────────────┐    │          │
│  │  │ AzureBastionSubnet (10.0.1.0/26)│    │          │
│  │  │                                 │    │          │
│  │  │  ┌──────────────────────────┐   │    │          │
│  │  │  │ Azure Bastion (Developer)│   │    │          │
│  │  │  │ - Public IP              │   │    │          │
│  │  │  └──────────────────────────┘   │    │          │
│  │  └─────────────────────────────────┘    │          │
│  └──────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────┘
```

## What is an Ephemeral OS Disk?

Ephemeral OS disks are stored on the local VM storage (cache or temp disk) rather than on Azure Storage, providing:

- 🚀 **Better Performance** - Lower read/write latency
- 💰 **Lower Cost** - No storage costs for the OS disk
- ⚡ **Faster Provisioning** - Quicker VM creation and reimaging
- 🗑️ **Automatic Cleanup** - Disk deleted when VM is deallocated

> **Note:** Data is NOT persisted when the VM is stopped/deallocated. Best for stateless workloads, dev/test, and containerized applications.

## What is Full Caching? (Public Preview)

**Announced:** March 30, 2026 | [Official Blog Post](https://techcommunity.microsoft.com/blog/AzureCompute/public-preview-ephemeral-os-disk-with-full-caching-for-vmvmss/4500191)

Traditionally, ephemeral OS disks store writes locally but still rely on remote storage for reads. **Full caching** eliminates this dependency by caching the **entire OS disk image** on local storage.

### Key Benefits:

- **🎯 Consistently Fast Performance** - Sub-millisecond latency for all OS disk operations
- **📈 10X Better IO Performance** - Compared to standard ephemeral OS disks
- **🛡️ Improved Resilience** - No impact during remote storage disruptions
- **⚡ Zero Boot Delay** - Caching happens asynchronously after VM boots
- **🔧 Local-Only Operations** - All OS disk I/O served from local storage

### How It Works:

1. VM boots normally using the OS image
2. Full OS disk image is cached to local storage in the background
3. Once complete, all OS disk reads/writes are served locally
4. Local storage capacity is reduced by **2× the OS disk size** to accommodate caching

### Ideal Workloads:

- ✅ **AI/ML Workloads** - High-performance OS disk for training and inference
- ✅ **Quorum-Based Databases** - Low-latency requirements for consistency
- ✅ **Data Analytics** - Real-time processing systems
- ✅ **Large-Scale Stateless Services** - Containerized workloads on General Purpose VMs
- ✅ **IO-Sensitive Applications** - Any workload requiring fast OS disk access

## Prerequisites

- Azure subscription
- Azure CLI 2.30+ or PowerShell Az module
- Contributor access to the resource group or subscription

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `vmName` | string | `vm-ephemeral` | Name of the virtual machine |
| `adminUsername` | string | *Required* | Administrator username for the VM |
| `adminPassword` | securestring | *Required* | Administrator password (must meet complexity requirements) |
| `location` | string | Resource Group location | Azure region for deployment |
| `vmSize` | string | `Standard_DS2_v2` | VM size (must support ephemeral disks) |
| `diskCaching` | string | `ReadWrite` | Caching type: `ReadOnly` or `ReadWrite` |
| `diffDiskPlacement` | string | `CacheDisk` | Placement: `CacheDisk` or `ResourceDisk` |

> **Full Caching Note:** This template uses `enableFullCaching: true` for enhanced performance (requires API version 2025-04-01 or later).

### Supported VM Sizes

- `Standard_DS2_v2`, `Standard_DS3_v2`
- `Standard_D2s_v3`, `Standard_D4s_v3`
- `Standard_E2s_v3`, `Standard_E4s_v3`

> Ensure the VM size has sufficient cache or temp disk space for the OS disk.

## Deployment

### Option 1: Deploy to Azure Button

Click the **Deploy to Azure** button at the top of this README. Update the URL with your GitHub username and repository name.

### Option 2: Azure CLI

```bash
# Create resource group
az group create \
  --name rg-ephemeral-vm \
  --location eastus

# Deploy with parameters file
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.json \
  --parameters vm-ephemeral.parameters.json \
  --parameters adminPassword='YourSecurePassword123!'

# Or deploy with inline parameters
az deployment group create \
  --resource-group rg-ephemeral-vm \
  --template-file vm-ephemeral.json \
  --parameters vmName='myvm' \
               adminUsername='azureadmin' \
               adminPassword='YourSecurePassword123!' \
               vmSize='Standard_DS2_v2' \
               location='eastus'
```

### Option 3: Azure PowerShell

```powershell
# Create resource group
New-AzResourceGroup -Name rg-ephemeral-vm -Location eastus

# Deploy template
New-AzResourceGroupDeployment `
  -ResourceGroupName rg-ephemeral-vm `
  -TemplateFile ./vm-ephemeral.json `
  -TemplateParameterFile ./vm-ephemeral.parameters.json `
  -adminPassword (ConvertTo-SecureString 'YourSecurePassword123!' -AsPlainText -Force)
```

### Option 4: Azure Portal

1. Navigate to **Create a resource** > **Template deployment (deploy using custom templates)**
2. Click **Build your own template in the editor**
3. Copy and paste the contents of `vm-ephemeral.json`
4. Click **Save**
5. Fill in the required parameters
6. Click **Review + create** > **Create**

## Connecting to Your VM

### Azure CLI (Recommended)

```bash
# SSH/RDP tunnel through Bastion
az network bastion rdp \
  --name <bastion-name> \
  --resource-group rg-ephemeral-vm \
  --target-resource-id <vm-resource-id>
```

The deployment outputs include a ready-to-use `connectCommand` with all values filled in.

### Azure Portal

1. Navigate to your VM in the Azure Portal
2. Click **Connect** > **Bastion**
3. Enter your username and password
4. Click **Connect**

## Outputs

| Output | Description |
|--------|-------------|
| `adminUsername` | The administrator username |
| `privateIPAddress` | Private IP address of the VM |
| `vmId` | Resource ID of the virtual machine |
| `bastionName` | Name of the Bastion host |
| `connectCommand` | Azure CLI command to connect via Bastion |

## Important Considerations

### Data Persistence

⚠️ **Ephemeral OS disks are reset when:**
- VM is stopped/deallocated
- VM is reimaged
- VM is deleted

Store important data on data disks, Azure Storage, or other persistent storage.

### Best Use Cases

**Optimal for Full Caching (Public Preview):**
- ✅ AI/ML training and inference workloads
- ✅ Quorum-based databases requiring low latency
- ✅ Real-time data analytics and processing
- ✅ IO-sensitive stateless services
- ✅ Large-scale containerized applications

**General Ephemeral OS Disk Use Cases:**
- ✅ Stateless applications
- ✅ Development and test environments
- ✅ Containerized workloads (Docker, Kubernetes nodes)
- ✅ CI/CD build agents
- ✅ Virtual Desktop Infrastructure (VDI)
- ✅ VM Scale Sets with golden images

### Limitations

- Cannot use Azure Backup directly on ephemeral OS disks
- VM must have sufficient cache or temp disk space
- Data is not persisted across deallocations
- **Full Caching:** Requires 2× OS disk size in local storage capacity
- **Full Caching:** Currently in Public Preview (availability expanding)

## Security Recommendations

- ✅ **No public IP on VM** (implemented)
- ✅ **Azure Bastion for secure access** (implemented)
- 🔐 Store passwords in Azure Key Vault
- 🔐 Use Azure AD authentication where possible
- 🔐 Enable Microsoft Defender for Cloud
- 🔐 Implement Just-In-Time (JIT) VM access
- 🔐 Use Managed Identities for Azure resource access

## Cost Optimization

**Estimated monthly cost (East US):**
- VM (Standard_DS2_v2): ~$70-100/month
- Bastion Developer SKU: ~$3.50/month
- Virtual Network: Free
- No storage costs for ephemeral OS disk ✅

> Actual costs may vary by region and usage. Check [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) for accurate estimates.

## Cleanup

```bash
# Delete the entire resource group and all resources
az group delete --name rg-ephemeral-vm --yes --no-wait
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Additional Resources

- 📢 [**Ephemeral OS Disk with Full Caching Announcement**](https://techcommunity.microsoft.com/blog/AzureCompute/public-preview-ephemeral-os-disk-with-full-caching-for-vmvmss/4500191) (March 30, 2026)
- [Ephemeral OS disks for Azure VMs](https://learn.microsoft.com/azure/virtual-machines/ephemeral-os-disks)
- [Azure Bastion documentation](https://learn.microsoft.com/azure/bastion/bastion-overview)
- [VM sizes in Azure](https://learn.microsoft.com/azure/virtual-machines/sizes)
- [Azure ARM template reference](https://learn.microsoft.com/azure/templates/)

## Support

If you encounter any issues or have questions:
- Open an [Issue](https://github.com/YOUR-USERNAME/YOUR-REPO/issues)
- Check existing [Discussions](https://github.com/YOUR-USERNAME/YOUR-REPO/discussions)
