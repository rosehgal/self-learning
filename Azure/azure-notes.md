# Microsoft Azure

## Types of Virtualization
> https://youtu.be/dckSc61WS6w

- Type 1
  - OS lies on hardware.
  - Virtualization software (hypervisor) is installed on the Host OS.
  - VMs on that computer are attached to the hypervisor.
  - VMs access hardware through hypervisor. Hypervisor will access OS for that functionality. OS will access the hardware.
  - **Disadvantages**
    - Slow performance
  - **Advantages**
    - Simple to install and run
    - Don't require special hardware/dedicated computer to run
  - **Examples**
    - Microsoft Virtual PC/Microsoft Virtual server
    - VMware workstation, fusion
    - VirtualBox

- Type 2
  - Hypervisor is install on the hardware. OS runs on hypervisor. Any VMs installed also run on hypervisor.
  - **Advantages**
    - Better performance
  - **Disadvantages**
    - Often require hardware features
      - E.g., Require virtualization to be supported in CPU and enabled in BIOS
  - **Examples**
    - Hyper-V
      - Advantage - run Windows and a virtualization solution
      - Sometimes mistaken as type 2 since an OS is required to run it.
        - E.g., For Windows Server 2012 R2, an HAL (hardware abstraction layer) provides communication between OS and hardware. HAL is replaced by Hyper-V, which has additional features for virtualization. So, even if OS is required, OS accesses the hardware just like a VM.
    - vSphere
      - Disadvantage - require a dedicated server to run the virtualization

## Azure Computes

### VM families
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes

Type | Description
-----|------------
General purpose | Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.
Compute optimized | High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.
Memory optimized | High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.
Storage optimized | High disk throughput and IO. Ideal for Big Data, SQL, and NoSQL databases.
GPU | Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs.
High performance compute | Our fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).

> Note: For Benchmark scores, refer https://docs.microsoft.com/en-us/azure/virtual-machines/linux/compute-benchmark-scores.

#### Azure Compute Unit (ACU)
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/acu

ACU provides a way of comparing compute (CPU) performance across Azure SKUs. This will help you easily identify which SKU is most likely to satisfy your performance needs.

### Maintenance and Updates
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/maintenance-and-updates

- Reboot-less Memory preserving maintenance
  - VM is paused for up to 30 seconds, preserving the memory in RAM, while the hosting environment applies the necessary updates and patches, or moves the VM to an already updated host.
  - Applied fault domain by fault domain.
  - Applications that perform real-time event processing, or high throughput networking scenarios are affected.

- Maintenance requiring reboot
  - Planned maintenance has two phases
    - self-service window
    - scheduled maintenance window
  - Availability Considerations during Scheduled Maintenance
    - Azure will only update the VMs in a single region of a region pair.
    - Only a single update domain is impacted at any given time.
      - Within an availability set, individual VMs are spread across up to 20 update domains (UDs).
      - Scale set (compute resource that enables you to deploy and manage a set of identical VMs as a single resource) is automatically deployed across UDs.

### Infrastructure guidelines
> Refer https://docs.microsoft.com/en-us/azure/virtual-machines/linux/infrastructure-example.

### Allocation failures
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/allocation-failure

- The servers in Azure datacenters are partitioned into clusters.
- Normally, an allocation request is attempted in multiple clusters.
- Sometimes, certain constraints from the allocation request force the Azure platform to attempt the request in only one cluster, referred to this as "pinned to a cluster".
- Allocation failures happen when an allocation request is pinned to a cluster, due to higher chance of failing to find free resources since the available resource pool is smaller.
  - Candidate cluster does not have free resources.
  - Candidate cluster does not support the requested VM size

Reason | Workaround
-------| ----------
Resize a VM or add VMs to an existing availability set | 1. Create a VM in a different availability set which can be added to the same vNet.<br> 2. Stop (deallocate) all VMs in the same availability set, then restart each one.
Restart partially stopped (deallocated) VMs | 1. Stop (deallocate) all VMs in the same availability set, then restart each one.
Restart fully stopped (deallocated) VMs | 1. If you use older VM series or sizes, consider moving to newer versions.<br> 2. Try another zone within the region that may have available capacity for the requested VM size.<br> 3. For large allocations, reduce the number of instances of the requested VM size, and then retry the deployment.<br> 4. For large allocations, use scale sets.

- Restarting VMs in a partially deallocated availability set is the same as adding VMs to an existing availability set.
- Stopping all VMs in the same availability set and restarting each one will make sure that a new allocation attempt is run and that a new cluster can be selected that has sufficient capacity.

### VM extensions and features

- Extensions are small applications that provide post-deployment configuration and automation tasks on Azure VMs.

#### Linux
> https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/features-linux

???

#### Windows
> https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/features-windows

???

## Azure Storage

### Introduction
> - https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction
> - https://docs.microsoft.com/en-us/azure/storage/common/storage-decide-blobs-files-disks

- Features
  - Durable and highly available
  - Secure
  - Scalable
  - Managed
  - Accessible

- Storage services

  - **Azure Blobs**: A massively scalable object store for text and binary (unstructured) data.
    - Serving images or documents directly to a browser.
    - Storing files for distributed access.
    - Streaming video and audio.
    - Storing data for backup and restore, disaster recovery, and archiving.
    - Storing data for analysis by an on-premises or Azure-hosted service.

  - **Azure Files**: Managed file shares for cloud or on-premises deployments.
    - Enables you to set up highly available network file shares accessible using SMB.
    - Can access the files from anywhere in the world using a URL that points to the file and includes a shared access signature (SAS) token.
    - The storage account credentials are used to provide authentication for access to the file share.

  - **Azure Queues**: A messaging store for reliable messaging between application components.
    - Queue messages can be up to 64 KB in size, and a queue can contain millions of messages.
    - Queues are generally used to store lists of messages to be processed asynchronously.

  - **Azure Tables**: A NoSQL store for schemaless storage of structured data.
    - Azure Cosmos DB Table API offering that provides throughput-optimized tables, global distribution, and automatic secondary indexes.

- **Disk storage** - Azure Storage also includes managed and unmanaged disk capabilities used by virtual machines.

- Each service is accessed through a storage account

#### Types of Storage Accounts

- General-purpose
  - Standard - use magnetic media to store data
  - Premium - use SSD to store data; high-performance storage for page blobs, which are primarily used for VHD files
- Blob
  - specialized storage account used to store block blobs and append blobs
  - can't store page blobs in these accounts, therefore can't store VHD files
  - allow you to set an access tier to Hot (frequently accessed) or Cool; the tier can be changed at any time

#### Accessing your blobs, files, and queues

- Each storage account has two authentication keys, either of which can be used for any operation.
- Keys can be rolled over.
- Ways to secure access
  - Using RBAC in Azure AD to control access to keys
    - only handles the management operations for that storage account like changing the access tier
    - can't grant access to data objects
    - can grant access to the storage account keys, which can then be used to read the data objects
  - Using SAS
    - a string containing a security token that can be attached to the URI for an asset that allows you to delegate access to specific storage objects and to specify constraints such as permissions and the date/time range of access.
- Public access to can be provided to a container and its blobs, or a specific blob.

#### Encryption

- Azure Storage Service Encryption (SSE) at rest automatically encrypts your data prior to persisting to storage and decrypts prior to retrieval.
- Client-side encryption libraries have methods you can call to programmatically encrypt data before sending it across the wire from the client to Azure.

#### Transferring data

- Use the AzCopy command-line utility to copy blob, and file data within your storage account or across storage accounts.
- Azure Import/Export service can be used to import or export large amounts of blob data to or from your storage account.
- To import large amounts of blob data to your storage account in a quick, inexpensive, and reliable way, you can also use Azure Data Box Disk.

### Disk Storage
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/about-disks-and-vhds

- All Azure VMs have at least two disks – a Linux OS disk and a temporary disk.
- The operating system disk is created from an image, and both the operating system disk and the image are virtual hard disks (VHDs) stored in an Azure storage account.
- VMS also can have one or more data disks, that are also stored as VHDs.
- Types
  - OS disk
    - Registered as SATA drive
    - Labelled as /dev/sda
    - 2 GB max capacity
  - Temp disk
    - /dev/sdb mounted to /mnt
  - Data disk
    - Registered as SCSI drive
    - 4 GB max capacity
- VHDs used in Azure are .vhd files stored as page blobs.
  - Before you can delete a source .vhd file, you’ll need to remove the lease by deleting the disk or image created from it.
  - All VHD files in Azure that you want to use as a source to create disks or images are read-only, except the .vhd files uploaded or copied to Azure storage by the user.
  - When you create a disk or image, Azure makes copies of the source .vhd files.
- Azure Disks are designed for 99.999% availability.
- 3 performance tiers for storage
  - Standard HDD
  - Standard SSD - only available as Managed Disks
  - Premium SSD
- Types of Disks
  - Managed
    - Ensures that you do not have to worry about the scalability limits of the storage account.
    - Can also manage your custom images in one storage account per Azure region, and use them to create hundreds of VMs in the same subscription.
  - Unmanaged
    - You have to figure out how to maximize the use of one or more storage accounts to get the best performance out of your VMs.

### Storage Replication
> https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy

- Options
  - Locally redundant storage (LRS)
  - Zone-redundant storage (ZRS)
  - Geo-redundant storage (GRS)
  - Read-access geo-redundant storage (RA-GRS)

### Scalability and Performance targets
> https://docs.microsoft.com/en-us/azure/storage/common/storage-scalability-targets

- When an application reaches the limit of what a partition can handle for the workload, Azure Storage begins to return error code 503 (Server Busy) or error code 500 (Operation Timeout) responses.
  - Application should use an exponential backoff policy for retries.
  - If the application exceeds the scalability targets of a single storage account, you can build your application to use multiple storage accounts.
- Check the [link](https://docs.microsoft.com/en-us/azure/storage/common/storage-scalability-targets) for information on targets.

### Using shared access signatures (SAS)
> https://docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1

- Allows to grant clients access to resources in your storage account, without sharing your account keys.
- Types of control
  - interval over which the SAS is valid
  - permissions granted by the SAS
  - optional IP address or range of IP addresses from which Azure Storage will accept the SAS
  - protocol over which Azure Storage will accept the SAS

???

> Note: Refer to this link to understand the [CAP theorem](https://dzone.com/articles/better-explaining-cap-theorem).
