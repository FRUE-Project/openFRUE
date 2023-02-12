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
/mnt/frue # wget -t=1o [PLACEHOLDER]
```
Now that you have downloaded the Stage 3, extract it.
```
/mnt/frue # tar xpvf *.tar.xz --numeric-owner --xattrs-include="*.*"
```
Now, it is a good idea to edit your *make.conf*. In this guide, we will use *vim*. But use whatever editor you want.
```
# vim /mnt/frue/etc/portage/make.conf
```
The *make.conf* file has most things preconfigured for optimized compiling. However, you should change the values of things like `VIDEO_CARDS="intel"`. Recommended changes are maked with a comment.
To exit vim and write changes, use `:wq!`

# Chapter 3! - Mount filesystems and chroot
In order to complete the openFRUE install, you must `chroot`
##### First, set up the ebuild repository.
```
# mkdir -p /mnt/frue/etc/portage/repos.conf
# cp /mnt/frue/usr/share/portage/config/repos.conf /mnt/frue/etc/portage/repos.conf/gentoo.conf
```
!! - if you get an error like "directory already exists", then skip this part.

##### Now, copy the DNS info
```
# cd /mnt/frue
/mnt/frue # cp --dereference /etc/resolv.conf etc/
```
##### Mount filesystems
```
/mnt/frue # mount --types proc /proc proc
/mnt/frue # mount --rbind /sys sys
/mnt/frue # mount --rbind /dev dev
/mnt/frue # mount --bind /run run
```
Now, its time to chroot!
```
/mnt/frue # chroot . /bin/bash
chroot # source /etc/profile
```
##### Mount the boot partition
You'll need to mount the boot partition in order to install a bootloader.
```
chroot # mount /dev/sda1 /boot
```
# Chapter 4! - Configuring Portage and other Portage shenanigans
##### Coppin' an ebuild repository from the web
If you are behind a firewall, or you can't use rsync for some reason, use `emerge-webrsync`:
```
chroot # emerge-webrsync
```
If you want to, it's wise to go ahead and set up a base to work with. To do this, `emerge` your *@world*:
```
chroot # emerge -avuDN @world
```
Out of the box, openFRUE is set with a /desktop/gnome profile. you can change this if you want to, but profile changes shouldn't be taken lightly, as it can break packages.
To change to a different profile:
```
chroot # eselect profile list

Available profile symlink targets:
  [1]   default/linux/amd64/17.1
  [2]   default/linux/amd64/17.1/desktop
  [3]   default/linux/amd64/17.1/desktop/gnome *
```
Remember, when changing profiles, make sure that it is version *17.1* and *does not* contain *systemd*.

##### CPU_FLAGS_*
Some ways of furthering optimization would include using CPU Flags.
emerge the package and echo to a use file:
```
chroot # emerge -av app-portage/cpuid2cpuflags
chroot # echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```
Out of the box, openFRUE comes with the `ACCEPT_LICENSE="*"` option in the make.conf.
If you wish to change it, edit the make.conf file and change it to how you like.
Example:
```
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
```
# Chapter 5! More configuring
##### Timezones
Now it is time to view and set timezones. To see available timezones:
```
chroot # ls /usr/share/zoneinfo/
```
Now that you have one you want to set, echo it to /etc/timezone and re-emerge timezone-data:
```
chroot # echo "Europe/Berlin" > /etc/timezone
chroot # emerge --config sys-libs/timezone-data
```
##### Locales
Use an editor to configure locales. It's recommended to have at least 1 UTF-8 locale.
```
chroot # vim /etc/locale.gen
```
Uncomment the locales you want, and then exit with `:wq!` and generate them:
```
chroot # locale-gen
```
Now, select the locale you want to use.
```
chroot # eselect locale list

Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   en_US
  [4]   en_US.utf8
  [ ]   (free form)
```
for this we're going to set *en_US.utf8* as our locale.
```
chroot # eselect locale set 4
```
Now, update your shell environment:
```
chroot # env-update ; source /etc/profile
```
# Chapter 6! Kernel configuration
Since openFRUE is so great, you can select which kernel that you want to use. For this guide, we are going to select *gentoo-kernel-bin* to make everything faster.
(And install *linux-firmware* as well, and microcode for Intel processors if necessary)
```
chroot # emerge -av sys-kernel/gentoo-kernel-bin
chroot # emerge -av sys-kernel/linux-firmware
chroot # emerge -av sys-firmware/intel-microcode
```
Once you have this all installed, select your kernel with *eselect*:
```
chroot # eselect kernel list

Available kernel symlink targets:
  [1]   linux-5.15-88-gentoo
```
```
chroot # eselect kernel set 1
```
##### The fstab file
An fstab is a very important file. openFRUE comes with *genfstab* out of the box, so it will be easier to make one. Create an fstab now:
```
chroot # genfstab -U / >> /etc/fstab
```
# Chapter 7! - More configuartion
##### Hostname
Now it's time to set up a hostname and install some other tools.
Setup your hostname:
```
chroot # vim /etc/conf.d/hostname
```
##### Enable networking startup services:
```
chroot # emerge -av net-misc/dhcpcd
chroot # rc-update add dhcpcd default
chroot # rc-update add NetworkManager default
chroot # rc-update add wpa_supplicant default
```
If rc-update complains about services missing, emerge their respective packages.

##### Root password and user creation
Creating a user and setting a root password is very important because, without these, you won't be able to use your openFRUE system.
Set root password:
```
chroot # passwd
```
##### Create user and assign necessities
```
chroot # useradd -c "Yue!" sudq # change "Yue!" and "sudq" to your own
chroot # passwd sudq
chroot # usermod -aG video,audio,plugdev,portage,users sudq
```
##### Change some boot configs
```
chroot # vim /etc/conf.d/keymaps
chroot # vim /etc/conf.d/hwclock
```
It's recommended to set `clock=local` if dualbooting with Windows.
# Chapter 8! Installing and configuring system tools
It's very important to install a system logger. For openFRUE, use *app-admin/sysklogd*
```
chroot # emerge -av app-admin/sysklogd ; rc-update add sysklogd default
```
Install *net-misc/chrony*, a way of sync'ing the system clock.
```
chroot # emerge -a net-misc/chrony ; rc-update add chronyd default
```
# Chapter 10! - Installing a bootloader and finalizing
##### GRUB
Congratulations on making it this far! You're almost ready to boot into your openFRUE system.
!! - `GRUB_PLATFORMS` should be configured in `make.conf` before continuing.
At this point, emerge *sys-boot/grub*
```
chroot # emerge -av sys-boot/grub
```
Once it's done, install GRUB to the boot partition.
```
chroot # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id="openFRUE"
```
or, for BIOS users,
```
chroot # grub-install /dev/sda --force
```
Now configure GRUB,
```
chroot # grub-mkconfig -o /boot/grub/grub.cfg
```
##### Disk cleanup
A very important way to test if your openFRUE install has been successful, run this command:
```
chroot # emerge cowsay ; cowsay fuck
```
Now, remove the tarball
```
chroot # rm /*.tar.xz
```
Now, here's the fun part. Exit the chroot and umount all filesystems, then reboot!
```
chroot # exit
# umount -R /mnt/frue
```
Now, reboot, and enjoy your new openFRUE computer! :cheez:
