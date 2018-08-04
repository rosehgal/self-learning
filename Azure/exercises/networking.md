# Networking

## Exercise 1

1. Create a Virtual Network and a subnet in Azure.
2. Create a Network Security Group that allows port 22 and port 80 and attach this NSG to the subnet.
3. Launch a Ubuntu 16.04 VM with Managed disk inside the VNet created earlier. Create a new NSG that allows port 22 and port 80 and attach it to the VM.
4. SSH into the VM and install apache web server in it. Try accessing the web page by navigating your browser to the public IP of the VM.
5. Remove the port 80 from the subnet NSG and try accessing the web page. Add the port 80 again to the subnet NSG.
6. Remove the port 80 from the VM NSG and try accessing the web page. Add the port 80 again to the VM NSG.

### Results

When port 80 is removed from the subnet NSG or VM NSG, the web page becomes inaccessible.

## Exercise 2

1. Create three Virutal Networks (say VNet A, B and C) with one subnet each. Make sure each VNet has a different CIDR range.
2. Launch a Ubuntu 16.04 VM with Managed disk inside each VNet created in the last step. SSH into VM and ping other VMs.
3. Create a VNet peering between VNet A and VNet B. Similarly, create another VNet peering between VNet B and VNet C.
4. Now VMs in VNet A and B should be able to ping each other. Likewise, VMs in B and C should be able to ping each other.
5. SSH into VM in VNet A and ping the VM in VNet C.
6. Create VNet peering between VNet A and VNet C. Repeat the step 5.

### Results

VMs in peered network can ping.

## Exercise 3

1. Create a Dynamic Public IP using the Azure portal and check if any IP is provided in your public IP resource.
2. Launch a Ubuntu 16.04 VM with Managed disk and attach the public IP created in the last step to this VM.
3. Now check the public IP resource again. Deallocate the VM and check the public IP resouce again. Delete all the resources.
4. Create a Static Public IP using the Azure portal and check if any IP is provided in your public IP resource.
5. Launch another Ubuntu 16.04 VM with Managed disk and attach the public IP created in the last step to this VM.
6. Now check the public IP resource again. Deallocate the VM and check the public IP resource again.

### Results

- Static public IP gets assigned at the time of creation of the public IP resource and is not lost even if the VM attached to it is deallocated. It also remains attached to the deallocated VM.
- Dynamic public IP gets assigned only when attached to a VM.
