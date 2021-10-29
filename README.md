# Arch Linux Installation & Invalid E: Error Debug Fix
Hello Everyone, this is a simplified instruction on how to install Arch Linux into a Virtual Machine and how to customize your Arch Linux VM. These instructions will also provide instructions on how to delete and re-install a virtual machine that is lost within your Flashdrive. 
# Invalid /E: when starting up VMware
Side note - This problem occured when I accidently wiped my hardive deleting all files including my VMware Workstation Pro. Upon re downloading VMware on my local computer an error would say "Invalid /E:" suggesting that VMware orignal file location could not be found since the external hardive was not recognized by my computer. The steps below will explain how to rename another USB to the original hardrive and trick the computer into recognize the USB.

Open up the command line and run it as Administrator. Then use 'diskpart' to start. Type 'list volume' and press enter. Select the volume you would like to assign, change or remove. To assign a new drive letter type 'assign letter=E'. After this you have named your new drive to the old drive that is either damaged or not recognized.
>diskpart
>
>select volume n
>
>assign letter=E

# Re-Installing VMware Workstation
Now that you have changed the disk name, you can re download vmware. I suggest changing the orignal filelocation back to your local drive as this will move the VM back into your local computer. Now create a new virtual machine and use Debian10.x64. However, since VMware does not automatically boot into UEFI mode you need to make changes to the VMX file.

On the second line of the vmx file write: 
>firmware = "efi"

Save the file and power on the virtual machine and choose UEFI and boot into your machine. 

# Install the ISO file 
### **Install from HTTP Direct Downloads**

Use the link below and download the file. Next, check the download image matches the checksum from m5sums.txt or sha1sums.txt as the other. Using GnuPG installed type this to insure the signature matches and verifies.
http://mirror.arizona.edu/archlinux/iso/2021.10.01/
>$ gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig




# Within the VM
### **Update System Clock**

Utilize timedatectl to update the system time
~~~
># timedatectl set npt-true
~~~
Next, check the status of your system time 
~~~
> # timedatectl status
~~~
If your system time zone is not correct type `# timedatectl set-timezone < your timezone` 

### **Partition the disks**
Use `# fdisk -l` to prompt all current partitions. 
Next, use `# cfdisk /dev/sda` to enter into your disk drive. You will then be able to see what names are assigned to each partition
Next, you need to create the first partition then select 'nvme0n1'. Most people will be given "sda1", however since I chose a larger disk (25G) size at the VMware installation, the naming scheme is different which is okay.

Press "Enter" for a new partition then type `500M`. Next open the select type menu and change the partition to `EFI 16/32/64`.

Next, create a new root parttion "nvm0n1p1" which will contain your file system. Give this partition `18G` for storange. This partition does not need to change. 

Finally, create the third and last partition "nvm0n1p2" and eneter `1G` of storage and make this partition a `Linux Swap`

**Configure the swap file**

Create the swap system
~~~
# mkswap /dev/nvmn1p2
# swapon /dev/nvm1p2
~~~

**Configure the root file**

Change the root file system as ext4
~~~
># mkfs.ext4 /dev/nvm0n1p1
~~~

**Configure the EFI file system**

Change the EFI file system  as a FAT32
~~~
mkfs.fat -F32 /dev/nvme0n1
~~~

**Mount the partitions**

Mount the root partition 
~~~
# mount /dev/nvm0n1p1 /mnt
~~~

Next, create a directory for the EFI partition and go ahead and mount it
~~~
# mkdir /mnt/boot 
# mount /dev/nvme0n1 /mnt/boot
~~~
 
# Installation

#### **Install the essential packages with** 

~~~
# pacstrap /mnt base linux linux-firmware
~~~

#### **Configure the system**

~~~
# genfstab -U /mnt >> /mnt/etc/fstab
~~~

#### **Change root into new system**

~~~
# arch-chroot /mnt
~~~

#### **Install a text editior while inside new system**

~~~
# pacman -S nano
~~~

### **Configurations for new Arch Linux**
### **Change Timezone**

~~~
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
~~~

#### **Set up HwClock**
~~~
# hwclock --systohc
~~~ 

**Set up Localization**
Use nano to modify `/etc/locale.gen` and uncomment "en_US.UTF-8 UTF-8"

Then create and modify a new file `/etc/locale.conf` to read:
~~~
LANG=en_US.UTF-8
~~~
**Set up Network Configuration**
create and modify the hostname file 
~~~
nano /etc/hostname
-------------------
myhostname
~~~

**Set the root password**
~~~
# passwd
~~~
**Install "GRUB" Bootloader**
pacman will be used to install the grub package and next we will install the boot loader onto the boot disk 
~~~
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
~~~

# QOL Changes
The next task is to install a Desktop Enviorment. This will allow the machine to be more User Friendly

**Add fish using pacman**
~~~
# pacman -S fish
~~~
**Make Fish shell configuration directory**

Add color to fish
~~~
mkdir -p ~/.config/fish
~~~ 
**Configure the file**
Now, its time to make the configuration file
~~~
nano ~/.config/fish/config.fish
~~~
**Install SSH**
We install ssh to have the ability to shell into the a gateway
~~~
# pacman -S openssh
~~~
# Install a Desktop Enviorment 
First we must install sudo and vim
~~~
# pacman -S sudo
# pacman -S vim
~~~
**Create two new users and set password**
~~~
# useradd -m -g users -s /bin/bash <username>
----------------------
# passwd
~~~
Next, add the new user to the sudoers list to give them sudo permissions
 ~~~~
 visudo 
 ~~~~
This will prompt you with the sudoers file. Scroll down and fine user "root". Then add the user and give them permissions 
~~~~
username ALL=(ALL) ALL
~~~~
**Install xorg graphics using pacman**
~~~
# pacman -S xorg-server xorg-apps xorg-xinit
~~~
**Install Graphics Server using pacman**
~~~
 # pacman -S xf86-video-vesa
~~~
**Install display manager using pacman**
 ~~~
 pacman -S sddm
 ~~~
 #Install Desktop Enviorment
 ~~~
 # pacman -S plasma kde-applications
 ~~~
 **Configure Graphical boot**
This allows the graphics driver to start on boot
~~~ 
# systemctl enable sddm
~~~
**Configure Network Manager**

Use pacman to install the networkmanager to allow the program to use have a userfriendly network managment program
~~~
# pacman -S networkmanager
# systemctl enable NetworkManager
~~~
# Rebooting
After all packages have been donwloaded. You would need to `exit` the chroot and `reboot`. If your reboot and startup page showes the username you created then everything should be correct. Now enter your password and enjoy your new customizable VM! 

- Andres Vargas 

















