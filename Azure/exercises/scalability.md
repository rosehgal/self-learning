# Scalability

## Exercise 1

1. Launch a Ubuntu 16.04 VM with Managed disk, SSH into the machine, install Apache web server and capture the machine as a VM Image.
2. Create a Virtual Machine scale set (VMSS) with the image created in the last step. Use "south india" as the location for your VM Scale set.
3. Also, use 2 as the instance count, "Standard_B1s" as the instance size and do not use managed disks for your scale set.
4. Let the autoscale be disabled and use load balancer to distribute your requests among VMs in your scale set.
5. Once your VM Scale set is ready, try accessing its load balancer public IP on your browser.
6. Delete one of your VMs in your scale set and check whether it is created again automatically.
7. If not, enable autoscaling in your scale set and set the scale mode to a specific instance count. Make sure your instance count is 2.
8. Repeat the step 6.

### Results

- Steps 2 to 4
> https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-use-custom-image-cli

  - To create a VMSS using the portal, click on "Browse all images" in the VMSS menu.
  - Unmanaged disk is not used since the image contains a managed disk. ???
  - Autoscale is disabled by default.
  - Configure the health probes and LB rules on the LB created, using the portal, if the VMSS is created using the CLI. The LB created is of SKU type _Basic_. It has the backend pools and inbound NAT rules configured for instances in the pool.

```bash
$ az vmss create -g scaleex1 -n my-vmss1 --image vm1-ubuntu-image-20180804232600 -l southindia --vm-sku Standard_B1s --lb vmss-lb2 \
>   --use-unmanaged-disk --storage-container-name new-stg-cont --storage-sku Standard_LRS
invalid usage for storage profile: create managed OS disk from custom image:
	not applicable: --storage-container-name, --use-unmanaged-disk
$ az vmss create -g scaleex1 -n my-vmss --image vm1-ubuntu-image-20180804232600 -l southindia --vm-sku Standard_B1s --lb vmss-lb1
```

- Step 6

  - VM is not automatically created.

- Step 8

  - VM is automatically created.

## Exercise 2

1. Launch a Ubuntu 16.04 VM with Managed disk, SSH into the machine, install Apache web server and capture the machine as a VM Image.
2. Create a Virtual Machine scale set with the image created in the last step and use "france central" as the location.
3. Choose all the availability zones, use 2 as the instance count, "Standard_B1s" as the instance size for your scale set.
4. Enable Autoscaling and keep 1 and 4 as the minimum and maximum number of VMs respectively
5. Choose load balancer to distribute the requests and launch your VM scale set.
6. Once VM scale set is created, change the cool down time to 5 minutes in both the scale out and scale in rules. Explore various metric sources, operators and operations available in scale rules.
7. Add your email address to the Scale set notification settings.
8. Launch a temporary Ubuntu 16.04 VM in the same VNet of the scale set instances. SSH into this temporary VM first and from there, SSH into the Scale set instances using their private IP address.
9. Install Stress tool and increase the CPU Utilization to hundred and check whether VMs are scaled out automatically. Stop the stress tool and check if VMs are scaled in automatically. Check your email inbox for scale set notification.
10. Try accessing the scale set load balancer's Public IP on your browser.

### Results

- Public IP of the Load balancer is of SKU type _Standard_.

**VMSS configuration**
![](/Azure/images/scale-ex2-vmss.png)

**Autoscaling configuration**
![](/Azure/images/scale-ex2-scaling.png)

**LB configuration**
![](/Azure/images/scale-ex2-lb.png)

**LB IP configuration**
![](/Azure/images/scale-ex2-pubip.png)

## Exercise 3

1. Create a Virtual Machine scale set with "south india" as the location using Azure CLI.
2. Use 2 as the instance count, "Standard_B1s" as the instance size, let the autoscale be disabled and use load balancer to distribute your requests among VMs in your scale set.
3. Using azure CLI, increase the instance count. Likewise, decrease the instance count.
4. Write a script that retrieves the CPU utilization metric of your VM Scale set and increases the Instance count by one, if this metric is greater than a specific value.
5. Similarly, it should decrease the instance count by one, if the CPU utilization metric is lesser than a given value.

### Results

- Step 1 and 2

  - Add health probes and rules to the LB via Azure portal.

```bash
az group create -n scaleex3 -l southindia
az vmss create -g scaleex3 -n vmss1si --image UbuntuLTS -l southindia --vm-sku Standard_B1s --lb vmss1si-lb1
```

![](/Azure/images/scale-ex3-lb.png)

- Step 3

```bash
az vmss scale -g scaleex3 -n vmss1si --new-capacity 3
```

```json
{
  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1si",
  "identity": null,
  "location": "southindia",
  "name": "vmss1si",
  "overprovision": true,
  "plan": null,
  "platformFaultDomainCount": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "scaleex3",
  "singlePlacementGroup": true,
  "sku": {
    "capacity": 3,
    "name": "Standard_B1s",
    "tier": "Standard"
  },
  "tags": {},
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "uniqueId": "eb7c6e5a-eb7c-483e-8e56-4650009a3114",
  "upgradePolicy": {
    "automaticOsUpgrade": false,
    "mode": "Manual",
    "rollingUpgradePolicy": null
  },
  "virtualMachineProfile": {
    "diagnosticsProfile": null,
    "evictionPolicy": null,
    "extensionProfile": null,
    "licenseType": null,
    "networkProfile": {
      "healthProbe": null,
      "networkInterfaceConfigurations": [
        {
          "dnsSettings": {
            "dnsServers": []
          },
          "enableAcceleratedNetworking": false,
          "enableIpForwarding": false,
          "id": null,
          "ipConfigurations": [
            {
              "applicationGatewayBackendAddressPools": null,
              "id": null,
              "loadBalancerBackendAddressPools": [
                {
                  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/loadBalancers/vmss1si-lb1/backendAddressPools/vmss1si-lb1BEPool",
                  "resourceGroup": "scaleex3"
                }
              ],
              "loadBalancerInboundNatPools": [
                {
                  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/loadBalancers/vmss1si-lb1/inboundNatPools/vmss1si-lb1NatPool",
                  "resourceGroup": "scaleex3"
                }
              ],
              "name": "vmss146e4IPConfig",
              "primary": null,
              "privateIpAddressVersion": "IPv4",
              "publicIpAddressConfiguration": null,
              "subnet": {
                "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/virtualNetworks/vmss1siVNET/subnets/vmss1siSubnet",
                "resourceGroup": "scaleex3"
              }
            }
          ],
          "name": "vmss146e4Nic",
          "networkSecurityGroup": null,
          "primary": true
        }
      ]
    },
    "osProfile": {
      "adminPassword": null,
      "adminUsername": "n0s0098",
      "computerNamePrefix": "vmss146e4",
      "customData": null,
      "linuxConfiguration": {
        "disablePasswordAuthentication": true,
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa AAAAB3...CF2+Q==\n",
              "path": "/home/n0s0098/.ssh/authorized_keys"
            }
          ]
        }
      },
      "secrets": [],
      "windowsConfiguration": null
    },
    "priority": null,
    "storageProfile": {
      "dataDisks": null,
      "imageReference": {
        "id": null,
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "16.04-LTS",
        "version": "latest"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "createOption": "FromImage",
        "image": null,
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        },
        "name": null,
        "osType": null,
        "vhdContainers": null,
        "writeAcceleratorEnabled": null
      }
    }
  },
  "zoneBalance": null,
  "zones": null
}
```
```bash
az vmss scale -g scaleex3 -n vmss1si --new-capacity 2
```
```json
{
  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Compute/virtualMachineScaleSets/vmss1si",
  "identity": null,
  "location": "southindia",
  "name": "vmss1si",
  "overprovision": true,
  "plan": null,
  "platformFaultDomainCount": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "scaleex3",
  "singlePlacementGroup": true,
  "sku": {
    "capacity": 2,
    "name": "Standard_B1s",
    "tier": "Standard"
  },
  "tags": {},
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "uniqueId": "eb7c6e5a-eb7c-483e-8e56-4650009a3114",
  "upgradePolicy": {
    "automaticOsUpgrade": false,
    "mode": "Manual",
    "rollingUpgradePolicy": null
  },
  "virtualMachineProfile": {
    "diagnosticsProfile": null,
    "evictionPolicy": null,
    "extensionProfile": null,
    "licenseType": null,
    "networkProfile": {
      "healthProbe": null,
      "networkInterfaceConfigurations": [
        {
          "dnsSettings": {
            "dnsServers": []
          },
          "enableAcceleratedNetworking": false,
          "enableIpForwarding": false,
          "id": null,
          "ipConfigurations": [
            {
              "applicationGatewayBackendAddressPools": null,
              "id": null,
              "loadBalancerBackendAddressPools": [
                {
                  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/loadBalancers/vmss1si-lb1/backendAddressPools/vmss1si-lb1BEPool",
                  "resourceGroup": "scaleex3"
                }
              ],
              "loadBalancerInboundNatPools": [
                {
                  "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/loadBalancers/vmss1si-lb1/inboundNatPools/vmss1si-lb1NatPool",
                  "resourceGroup": "scaleex3"
                }
              ],
              "name": "vmss146e4IPConfig",
              "primary": null,
              "privateIpAddressVersion": "IPv4",
              "publicIpAddressConfiguration": null,
              "subnet": {
                "id": "/subscriptions/0bb3891d-28db-464e-8daf-cb07d9d312d2/resourceGroups/scaleex3/providers/Microsoft.Network/virtualNetworks/vmss1siVNET/subnets/vmss1siSubnet",
                "resourceGroup": "scaleex3"
              }
            }
          ],
          "name": "vmss146e4Nic",
          "networkSecurityGroup": null,
          "primary": true
        }
      ]
    },
    "osProfile": {
      "adminPassword": null,
      "adminUsername": "n0s0098",
      "computerNamePrefix": "vmss146e4",
      "customData": null,
      "linuxConfiguration": {
        "disablePasswordAuthentication": true,
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa AAAAB3...CF2+Q==\n",
              "path": "/home/n0s0098/.ssh/authorized_keys"
            }
          ]
        }
      },
      "secrets": [],
      "windowsConfiguration": null
    },
    "priority": null,
    "storageProfile": {
      "dataDisks": null,
      "imageReference": {
        "id": null,
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "16.04-LTS",
        "version": "latest"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "createOption": "FromImage",
        "image": null,
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        },
        "name": null,
        "osType": null,
        "vhdContainers": null,
        "writeAcceleratorEnabled": null
      }
    }
  },
  "zoneBalance": null,
  "zones": null
}
```

- Step 4 and 5
> https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-autoscale-cli

  - To check the CPU utilization metric, run  
  `az monitor metrics list --resource /subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/ABC/providers/Microsoft.Compute/virtualMachineScaleSets/<vmss_name> --metric "Percentage CPU" --interval PT30M`

  - Script to auto scale

```bash
#!/bin/bash -x

# Enable autoscaling
az vmss update -g SCALEEX3 -n vmss1si --set upgradePolicy.mode=automatic

# Create autoscale settings
az monitor autoscale create -g SCALEEX3 \
  --resource vmss1si \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale1 \
  --min-count 1 \
  --max-count 4 \
  --count 2

# Scale out rule
az monitor autoscale rule create -g SCALEEX3 \
 --autoscale-name autoscale1 \
 --condition "Percentage CPU > 70 avg 5m" \
 --scale out 1

# Scale in rule
az monitor autoscale rule create -g SCALEEX3 \
 --autoscale-name autoscale1 \
 --condition "Percentage CPU < 30 avg 5m" \
 --scale in 1
```

![](/Azure/images/scale-ex3.png)
