# Compute

## Exercise 1

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with managed disk using the Azure Portal.
2. Stop the VM and observe the status. It should automatically get deallocated.
3. Install Azure CLI on your machine if possible. Else use Azure CLI available in the Azure cloud shell.
4. Using Azure CLI, create a VM, stop it and check its status using CLI as well as the portal. It will be in a stopped state but not deallocated. Observe if anything else is different compared to the machine in the deallocated state.
5. Create an another VM and SSH into it. Run the "shutdown now" command and check the state of the VM. Delete all the resources.

### Results

- Commands used

```bash
$ az vm create -n trial-eastus-vmUbuntu16-4  -g ns-training --image UbuntuLTS
{
  "fqdns": "",
  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/ns-training/providers/Microsoft.Compute/virtualMachines/trial-eastus-vmUbuntu16-4",
  "location": "eastus",
  "macAddress": "00-0D-3A-19-BA-FB",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.6",
  "publicIpAddress": "137.135.103.33",
  "resourceGroup": "ns-training",
  "zones": ""
}
$ az vm stop -n trial-eastus-vmUbuntu16-3 -g NS-TRAINING
{
  "endTime": "2018-08-02T11:11:59.819924+00:00",
  "error": null,
  "name": "b60a5a67-88d9-465e-bf9d-65e0aec0afb1",
  "startTime": "2018-08-02T11:11:45.179298+00:00",
  "status": "Succeeded"
}
$ az vm deallocate -n trial-eastus-vmUbuntu16-4 -g NS-TRAINING
{
  "endTime": "2018-08-02T11:14:54.355259+00:00",
  "error": null,
  "name": "50e62574-eda4-44f9-a736-08e8de16ca6c",
  "startTime": "2018-08-02T11:12:29.366987+00:00",
  "status": "Succeeded"
}
```

![](/Azure/images/compute-ex1.png)

![](/Azure/images/compute-ex1-2.png)

## Exercise 2

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with managed disk using the Azure Portal and SSH into the VM.
2. Install Apache server and Capture the VM as an image. Observe the steps shown in the Azure portal while capturing the VM as an image. Then delete the VM.
3. Repeat the step 1 and Install Apache server in the VM. Then deprovision the machine and eliminate machine related information from the VM with the help of azure agent inside the machine.
4. Using Azure CLI, Deallocate the VM and Generalize it for capturing as an image. Finally Capture the machine as an image using Azure CLI.
5. Try starting the Generalized VM and observe the error. Delete the VM and use the Deleted VM's boot disk to launch another VM.
6. Similarly create a Windows VM, put some files and try capturing the VM as an image using Azure CLI.

### Results

- Step 1 and 2
![](/Azure/images/compute-ex2-step1-step2.png)

- Step 3

```bash
n0s0098@trial-centus-vmUbuntu16-2:~$  # This is in the VM
n0s0098@trial-centus-vmUbuntu16-2:~$  sudo waagent -deprovision+user
WARNING! The waagent service will be stopped.
WARNING! Cached DHCP leases will be deleted.
WARNING! root password will be disabled. You will not be able to login as root.
WARNING! /etc/resolvconf/resolv.conf.d/tail and /etc/resolvconf/resolv.conf.d/original will be deleted.
WARNING! n0s0098 account and entire home directory will be deleted.
Do you want to proceed (y/n)y
```

- Step 4

```bash
$  az vm deallocate -g NS-COMPUTE-EX2 -n trial-centus-vmUbuntu16-2
{
  "endTime": "2018-08-02T12:31:26.572670+00:00",
  "error": null,
  "name": "fcd2d425-caa6-487b-8d5a-e4f08eec32de",
  "startTime": "2018-08-02T12:29:03.604939+00:00",
  "status": "Succeeded"
}
$  az vm generalize -g NS-COMPUTE-EX2 -n trial-centus-vmUbuntu16-2
$  az vm capture -g NS-COMPUTE-EX2 -n trial-centus-vmUbuntu16-2 --vhd-name-prefix myGenImage-fromCapture
The Capture action is only supported on a Virtual Machine with blob based disks. Please use the 'Image' resource APIs to create an Image from a managed Virtual Machine.
$  az image create -g NS-COMPUTE-EX2 -n myGenImage --source trial-centus-vmUbuntu16-2
{
  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/NS-COMPUTE-EX2/providers/Microsoft.Compute/images/myGenImage",
  "location": "centralus",
  "name": "myGenImage",
  "provisioningState": "Succeeded",
  "resourceGroup": "NS-COMPUTE-EX2",
  "sourceVirtualMachine": {
    "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/NS-COMPUTE-EX2/providers/Microsoft.Compute/virtualMachines/trial-centus-vmUbuntu16-2",
    "resourceGroup": "NS-COMPUTE-EX2"
  },
  "storageProfile": {
    "dataDisks": [],
    "osDisk": {
      "blobUri": null,
      "caching": "ReadWrite",
      "diskSizeGb": 30,
      "managedDisk": {
        "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/ns-compute-ex2/providers/Microsoft.Compute/disks/trial-centus-vmUbuntu16-2_OsDisk_1_ed7c7514c02949278879dd96d11b2917",
        "resourceGroup": "ns-compute-ex2"
      },
      "osState": "Generalized",
      "osType": "Linux",
      "snapshot": null,
      "storageAccountType": "Standard_LRS"
    },
    "zoneResilient": null
  },
  "tags": {},
  "type": "Microsoft.Compute/images"
}
```

- Step 5

```bash
$  az vm start -g NS-COMPUTE-EX2 -n trial-centus-vmUbuntu16-2
Operation 'Start VM' is not allowed on VM 'trial-centus-vmUbuntu16-2' since the VM is generalized.
$  az vm delete -y -g NS-COMPUTE-EX2 -n trial-centus-vmUbuntu16-2
{
  "endTime": "2018-08-02T12:52:39.144299+00:00",
  "error": null,
  "name": "cbdca0de-d515-4e55-a2bd-2691e0bb7a3e",
  "startTime": "2018-08-02T12:52:28.621027+00:00",
  "status": "Succeeded"
}
```

![](/Azure/images/compute-ex2-step5.png)

- Step 6

```
az vm deallocate  -g NS-COMPUTE-EX2 -n centus-vm-4
az vm generalize -g NS-COMPUTE-EX2 -n centus-vm-4
az image create -g NS-COMPUTE-EX2 --source centus-vm-4 -n myWindowsImage
```

## Exercise 3

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with managed disk using the Azure Portal and SSH into the VM.
2. Using Azure CLI, try resizing the machine to an another size like "Standard_B1ms".
3. Try resizing the machine using the Azure portal.
4. If the VM is stopped before resizing, your SSH connection will get terminated.

### Results

- Commands used

```
az vm resize --size Standard_B1ms  -g ex3 -n newVm
```
