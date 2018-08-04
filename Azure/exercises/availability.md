# Availability

## Exercise 1

1. Launch a Ubuntu 16.04 VM with Managed disk.
2. Attach one more network interface card to the VM.
3. SSH into the VM and install apache web server.
4. Try accessing the public IPs of both the network interface cards attached to the VM.
5. Create a Basic Load Balancer and add this VM to the load balancer.
6. Add both the Network Interface cards as the target network configuration to the LB.
7. Access the Load balancer's IP through the browser and validate that it is distributing the requests to various Network interfaces of a virtual machine.

### Results
> https://docs.microsoft.com/en-us/azure/load-balancer/quickstart-create-basic-load-balancer-portal

- Step 2

  - Create a NIC, and enable public IP address settings in it.
  - Add a NSG to the NIC. You can add the existing NSG that was created with the Ubuntu VM for port 80 and 22.
  - To attach it to the VM, deallocate the VM, attach the NIC, and start the VM again.

- Step 3

  - Create 2 web servers on 2 different interfaces using `python3 -m http.server --bind <w.x.y.z> 80`.

```bash
root@vm1:~# # inside the VM
root@vm1:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0d:3a:19:3a:71 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fe19:3a71/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0d:3a:19:97:3c brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fe19:973c/64 scope link
       valid_lft forever preferred_lft forever
root@vm1:~# curl 10.0.0.4
nic1 default 10.0.0.4
root@vm1:~# curl 10.0.0.5
nic2 manmade 10.0.0.5
```

- Step 4

  - The public IP on the second NIC is not accessible from the internet even after adding NSG rules.

    - The reason for this is stated in this Azure [doc](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface-vm) as:
    > Though you can control which network interface you sent outbound traffic to, by default, all outbound traffic from the VM is sent out the IP address assigned to the primary IP configuration of the primary network interface.

    - To circumvent this issue, enter the following commands in the VM to add routes for outbound connectivity from the secondary NIC (Reference [link](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#validation-linux)).

```bash
$ # echo 150 <routing_table_name> >> /etc/iproute2/rt_tables
$   echo 150 custom >> /etc/iproute2/rt_tables
$ # ip rule add from <NIC_private_IP> lookup <routing_table_name>
$   ip rule add from 10.0.0.5 lookup custom
$ # ip route add default via <gateway> dev <NIC_name> table <routing_table_name>
$   ip route add default via 10.0.0.1 dev eth1 table custom
```

![](/Azure/images/aval-ex1-1.png)

![](/Azure/images/aval-ex1-2.png)

- Step 6

  - Add the VM and it's 2 NICs as a backend pool.
  - Add a health probe.
  - Add a load balancing rule.

## Exercise 2

1. Create a new Availability Set with at least 1 fault domain and 1 update domain.
2. Launch two Ubuntu 16.04 VMs with Managed disk in the availability set created earlier.
3. SSH into both VMs and install apache web server.
4. Create a Basic Load Balancer and then add the created availability set as the backend pool.
5. Access the Load balancer's IP through the browser and check whether it is distributing the requests to both the virtual machines.

### Results

- Step 4

  - Add the availability set as a backend pool.
  - Add a health probe.
  - Add a load balancing rule.

## Exercise 3

1. Launch two Ubuntu 16.04 VMs with Managed disk.
2. SSH into both VMs and install apache web server.
3. Change the SKU property of Public IPs to "standard".
4. Create a standard Load Balancer and then add the Virtual Machines to the backed pool.
5. Access the Load balancer's IP through the browser and check whether it is distributing the requests to both the virtual machines.

### Results

- Step 3

  - Trying to change the SKU property shows that it is not possible. ???
  - Create new VMs with public IPs' SKU set to standard.

```bash
$ az network public-ip update --sku Standard -n vm3-ip -g avex3
Standard sku publicIp /subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/avex3/providers/Microsoft.Network/publicIPAddresses/vm3-ip must have AllocationMethod set to Static.
$ az network public-ip update --allocation-method Static --sku Standard -n vm3-ip -g avex3
Sku property is set at creation time and cannot be changed from Basic to Standard on resource update for resource /subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/avex3/providers/Microsoft.Network/publicIPAddresses/vm3-ip.
```

- Step 4

  - Add backend pool, health probe and LB rule.

![](/Azure/images/aval-ex3-bepool.png)
![](/Azure/images/aval-ex3-hp.png)
![](/Azure/images/aval-ex3-lbrule.png)
![](/Azure/images/aval-ex3-summary.png)
