# Storage

## Exercise 1

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with an unmanaged disk using the Azure Portal.
2. Once the VM is created, convert your unmanaged disk to a managed disk.
3. Check the storage account whether the VHD of the unmanaged disk still exist.
4. Delete all the created resources.

### Results

- Step 2

  - Go to Storage accounts > [your storage account] > Overview > Services - Blobs > vhds
  - Go to Virtual machines > [your VM] > Disks > [your disk]

The vhd listed in the above 2 steps should be the same.

![](/Azure/images/storage-ex1.png)

## Exercise 2

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with a managed disk using the Azure Portal.
2. Create and attach a data disk of size 2 GB to the VM.
3. SSH into the VM, format the newly attached data disk to an ext4 filesystem and mount it to a new directory.
4. Write some Hello World files in to the new disk as well the temporary disk available (mounted by default).
5. Capture the VM as an image, delete the VM and create a new VM with the image created earlier.
6. SSH into the VM and check whether the data disks exist, formatted and has the file we created earlier.
7. Detach the data disk, resize it to 4 GB and attach it back to the VM.
8. SSH again and check the size of the disk and the contents inside it.
9. Also detach the disk again and try resizing it to 2 GB.

### Results

- Step 3
> Reference: https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

  1. Use the commands `parted -l` or `lsblk` to look for disks without partitions.
  2. Choose GPT partitioning standard, `parted /dev/sdc mklabel gpt`.
  3. Create the partition, `parted -a opt /dev/sdc mkpart primary ext4 0% 100%`.
  4. Format to EXT4 filesystem, `mkfs.ext4 -L datapartition /dev/sdc1`.
  5. Create a mount directory, `mkdir -p /mnt/data`.
  6. Mount the filesystem temporarily, `mount -o defaults /dev/sdc1 /mnt/data`.
  7. Test the mount, `df -h -x tmpfs -x devtmpfs`.

```bash
n0s0098@ubuntu-vm-1:~$ sudo su -
root@ubuntu-vm-1:~# parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Error: /dev/sdc: unrecognised disk label
Model: Msft Virtual Disk (scsi)                                           
Disk /dev/sdc: 2147MB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags:

root@ubuntu-vm-1:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-1:~# parted /dev/sdc mklabel gpt
Information: You may need to update /etc/fstab.

root@ubuntu-vm-1:~#  parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Model: Msft Virtual Disk (scsi)
Disk /dev/sdc: 2147MB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags


root@ubuntu-vm-1:~#  lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-1:~# parted -a opt /dev/sdc mkpart primary ext4 0% 100%
Information: You may need to update /etc/fstab.

root@ubuntu-vm-1:~# lsblk -fs                                             
NAME  FSTYPE LABEL           UUID                                 MOUNTPOINT
sdb1  ext4                   b63d35e1-d701-4c74-937f-a09167d5714e /mnt
└─sdb                                                             
sr0                                                               
sdc1                                                              
└─sdc                                                             
sda1  ext4   cloudimg-rootfs cdf895f5-190c-422a-b9a5-e8ba4cfad807 /
└─sda                                                             
root@ubuntu-vm-1:~#  parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Model: Msft Virtual Disk (scsi)
Disk /dev/sdc: 2147MB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB               primary


root@ubuntu-vm-1:~#  lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
└─sdc1   8:33   0    2G  0 part
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-1:~# mkfs.ext4 -L datapartition /dev/sdc1
mke2fs 1.42.13 (17-May-2015)
Discarding device blocks: done                            
Creating filesystem with 523776 4k blocks and 131072 inodes
Filesystem UUID: 8ff7f540-26a5-4204-9ccc-d4388b86fda5
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-vm-1:~#  lsblk -fs
NAME  FSTYPE LABEL           UUID                                 MOUNTPOINT
sdb1  ext4                   b63d35e1-d701-4c74-937f-a09167d5714e /mnt
└─sdb                                                             
sr0                                                               
sdc1  ext4   datapartition   8ff7f540-26a5-4204-9ccc-d4388b86fda5
└─sdc                                                             
sda1  ext4   cloudimg-rootfs cdf895f5-190c-422a-b9a5-e8ba4cfad807 /
└─sda                                                             
root@ubuntu-vm-1:~#  parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Model: Msft Virtual Disk (scsi)
Disk /dev/sdc: 2147MB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB  ext4         primary


root@ubuntu-vm-1:~#  lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
└─sdc1   8:33   0    2G  0 part
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-1:~# cat /etc/fstab
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
UUID=cdf895f5-190c-422a-b9a5-e8ba4cfad807	/	 ext4	defaults,discard	0 0
/dev/disk/cloud/azure_resource-part1	/mnt	auto	defaults,nofail,x-systemd.requires=cloud-init.service,comment=cloudconfig	0	2
root@ubuntu-vm-1:~# mkdir -p /mnt/data
root@ubuntu-vm-1:~# mount -o defaults /dev/sdc1 /mnt/data
root@ubuntu-vm-1:~#  parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Model: Msft Virtual Disk (scsi)
Disk /dev/sdc: 2147MB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB  ext4         primary


root@ubuntu-vm-1:~#  lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
└─sdc1   8:33   0    2G  0 part /mnt/data
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-1:~# df -h -x tmpfs -x devtmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  1.4G   28G   5% /
/dev/sdb1       3.9G  8.0M  3.7G   1% /mnt
/dev/sdc1       2.0G  3.0M  1.9G   1% /mnt/data
```

- Step 4

```bash
cd / && echo 'Hello World from disk sda/sda1 30G mount /' > mydata-pri.dat
cd /mnt && echo 'Hello World from disk sdb/sdb1 4G mount /mnt' > mydata-temp.dat
cd /mnt/data && echo 'Hello World from disk sdc/sdc1 2G mount /mnt/data' > mydata-data.dat
```

- Step 6

```bash
root@ubuntu-vm-2-fromimage:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
└─sdc1   8:33   0    2G  0 part
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-2-fromimage:~# lsblk -fs
NAME  FSTYPE LABEL           UUID                                 MOUNTPOINT
sdb1  ext4                   3f212b30-695d-4146-8535-6781768f99cd /mnt
└─sdb                                                             
sr0                                                               
sdc1  ext4   datapartition   8ff7f540-26a5-4204-9ccc-d4388b86fda5
└─sdc                                                             
sda1  ext4   cloudimg-rootfs cdf895f5-190c-422a-b9a5-e8ba4cfad807 /
└─sda                                                             
root@ubuntu-vm-2-fromimage:~# cat /mydata-pri.dat
Hello World from disk sda/sda1 30G mount /
root@ubuntu-vm-2-fromimage:~# ls /mnt/
DATALOSS_WARNING_README.txt  lost+found
root@ubuntu-vm-2-fromimage:~# mkdir -p /mnt/data
root@ubuntu-vm-2-fromimage:~# mount -o defaults /dev/sdc1 /mnt/data
root@ubuntu-vm-2-fromimage:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sdc      8:32   0    2G  0 disk
└─sdc1   8:33   0    2G  0 part /mnt/data
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-2-fromimage:~# ls /mnt/data/
lost+found  mydata-data.dat
root@ubuntu-vm-2-fromimage:~# cat /mnt/data/mydata-data.dat
Hello World from disk sdc/sdc1 2G mount /mnt/data
```

- Step 7

  - Go to Virtual machines > [Your VM] > Disks, click on Edit and Detach the Data disk.
  - Go to Disks > [the detached 2GB data disk] and resize.
  - Go to Virtual machines > [Your VM] > Disks and add the resized disk.

**Before resize**
![](/Azure/images/storage-ex2-before.png)

**After resize**
![](/Azure/images/storage-ex2-resize.png)

- Step 8

```bash
root@ubuntu-vm-2-fromimage:~# lsblk # after detaching
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-2-fromimage:~# lsblk # after re-attaching
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdd      8:48   0    4G  0 disk
└─sdd1   8:49   0    2G  0 part
sdb      8:16   0    4G  0 disk
└─sdb1   8:17   0    4G  0 part /mnt
sr0     11:0    1  628K  0 rom  
sda      8:0    0   30G  0 disk
└─sda1   8:1    0   30G  0 part /
root@ubuntu-vm-2-fromimage:~# lsblk -fs
NAME  FSTYPE LABEL           UUID                                 MOUNTPOINT
sdd1  ext4   datapartition   8ff7f540-26a5-4204-9ccc-d4388b86fda5
└─sdd                                                             
sdb1  ext4                   3f212b30-695d-4146-8535-6781768f99cd /mnt
└─sdb                                                             
sr0                                                               
sda1  ext4   cloudimg-rootfs cdf895f5-190c-422a-b9a5-e8ba4cfad807 /
└─sda                                                             
root@ubuntu-vm-2-fromimage:~#  ls /mnt/data/
ls: reading directory '/mnt/data/': Input/output error
root@ubuntu-vm-2-fromimage:~# parted -l
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  32.2GB  32.2GB  primary  ext4         boot


Model: Msft Virtual Disk (scsi)
Disk /dev/sdb: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      65.5kB  4294MB  4294MB  primary  ext4


Warning: Not all of the space available to /dev/sdd appears to be used, you can
fix the GPT to use all of the space (an extra 4194304 blocks) or continue with
the current setting?
Fix/Ignore? I                                                             
Model: Msft Virtual Disk (scsi)
Disk /dev/sdd: 4295MB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB  ext4         primary


root@ubuntu-vm-2-fromimage:~# mount -o defaults /dev/sdd1 /mnt/data
root@ubuntu-vm-2-fromimage:~# ls /mnt/data/
lost+found  mydata-data.dat
root@ubuntu-vm-2-fromimage:~# cat /mnt/data/mydata-data.dat
Hello World from disk sdc/sdc1 2G mount /mnt/data
```

- Step 9

![](/Azure/images/storage-ex2-downsize.png)

## Exercise 3

1. Create two Ubuntu 16.04 VM of size "Standard_B1s" with a managed disk using the Azure Portal.
2. Create a new storage account and create a new file share of quota 2 GB in it.
3. Mount the file share to all your Ubuntu VMs.
4. Create a file in the VM1 and check whether the file is available using the VM2.
5. Try resizing the quota of your file share and check whether your VMs get affected.
6. Also, try uploading some files using your Azure portal and take a snapshot of your file share.
7. Delete all the files in your file share using your VM and restore them back using the snapshot taken in the last step.

### Results

- Step 3

![](/Azure/images/storage-ex3.png)

```bash
root@ubuntu-vm-1:/mnt/data# df -h -x tmpfs -x devtmpfs
Filesystem                                        Size  Used Avail Use% Mounted on
/dev/sda1                                          30G  1.4G   28G   5% /
/dev/sdb1                                         3.9G  8.0M  3.7G   1% /mnt
//ns0stacc1.file.core.windows.net/file-share-2gb  2.0G   64K  2.0G   1% /mnt/data
```

- Step 5

```bash
root@ubuntu-vm-2:/mnt/data# df -h -x tmpfs -x devtmpfs
Filesystem                                        Size  Used Avail Use% Mounted on
/dev/sda1                                          30G  1.4G   28G   5% /
/dev/sdb1                                         3.9G  8.0M  3.7G   1% /mnt
//ns0stacc1.file.core.windows.net/file-share-2gb  3.0G   64K  3.0G   1% /mnt/data
```

- Step 7

![](/Azure/images/storage-ex3-2.png)

## Exercise 4

1. Create a new storage account of kind v1 and create a new blob container with anonymous blob read access.
2. Create a new file "index.html" with some text and upload it to the container created in the last step.
3. Copy the generated URL and open it using the browser.
4. Create a new storage account of kind v2 and enable static web site in it.
5. Provide "index.html" as the index document name and upload your index.html file to the $web container.
6. Access the primary endpoint of your static website through the browser.

### Results

- Step 5

  - Go to Storage accounts > [your storage account] > Static website > $web and upload the files.
