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
