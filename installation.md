# openFRUE Installation Guide
This guide was written to provide an easy to follow tutorial on how to install a GNOME openFRUE system.

# About the install
openFRUE was created to combine the chuckle'ish ways of FRUE, with the power and simplicity of Gentoo Linux.
This guide will teach you how to install the operating system, with the GRUB2 bootloader, and a preconfigured GNOME environment.

# Chapter 1! - Networking
Networking is a very important piece of the openFRUE install. If you are using a LiveCD with a GUI, you can enable networking with their respective menus.
However, if you are using something like an Arch Linux or Gentoo Linux LiveCD, you can enable networking like so:
### Arch CD:
- ```# iwctl```
- ```iwctl# device list``` (your devices may look different. At the time of writing, this machine used wlan0 was the wifi card.)
- ```iwctl# station wlan0 get-networks```
- ```iwctl# station wlan0 connect SSID``` (change SSID to the name of your wifi network)
- ```iwctl# exit```
### Gentoo CD:
- ```# net-setup``` (this will pull up a menu for you to select your device, type your SSID, and the network password)

# Chapter 2! - Partitioning & other disk-related shenanigans
Partitioning is the art of cuttin' up your hard drive into some slices so that everything is organized.
To save time, I'm just gonna put how to do partitioning for UEFI.
### fdisk:
##### Format disk, and create EFI partition
```
Command (m for help): g
Created a new GPT disklabel (GUID:some numbers n shit)
```
```
Command (m for help): n
Parition number (1-128, default 1): 1
First sector (2048-60549086, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-60549086, default 60549086): +256M

Created a new partition 1 of type 'Linux filesystem' and a size of 256 MiB.
```
```
Command (m for help): t
Selected partition 1
Partition type (type L to list all types): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.
```
##### Create swap partition (This will create a 4GB swap, set to how much you want)
```
Command (m for help): n
First sector (526336-60549086, default 526336):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (526336-60549086, default 60549086): +4G

Created a new partition 2 of type 'Linux filesystem' and of size 4 GiB.
```
```
Command (m for help): t
Partition number (1,2, default 2): 2
Partition type (type L to list all types): 19

Changed type of partition 'Linux filesystem' to 'Linux swap'.
```
##### Create root partition
- When prompted, just press `Enter` for everything, to use the rest of the space for the root partition.

##### Save partition layout
```
Command (m for help): w
```

Now, time to format these partitions and give them a filesystem
```
# mkfs -F 32 /dev/sda1
# mkswap /dev/sda2
# swapon /dev/sda2
# mkfs.ext4 /dev/sda3
```
The first command makes /dev/sda1 a vfat filesystem, the second and third make /dev/sda2 a swap device, and the third makes /dev/sda3 an ext4 filesystem.

#### Mounting the root partition
When using a LiveCD, you will need to make a mount directory for the new openFRUE system.
```
# mkdir -p /mnt/frue
# mount /dev/sda3 /mnt/frue
```
##### Now that your / is mounted, `cd` to /mnt/frue
```
# cd /mnt/frue
```
At this point, we are going to now download a Stage3 for the new system. This includes all packages necessary for an install.
To download a Stage3, use `wget`:
```
# wget -t=1o [PLACEHOLDER]
```
this guide is not finished yet
