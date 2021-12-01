# Installing Void-linux
This is my try to install Void-linux with full disk encryption, specifically the x86_64 version with glibc as system clib, on a Secure Boot system, from a Manjaro KDE live usb because even without a PK key (it does not have a separate "disable Secure Boot" option) my motherboard firmware does not seem to allow booting from the Void live usb. Manjaro is an Arch based distro, and as such provide the very useful AUR package for the Void-linux package handler, XBPS.
## Preparations
Before we can even try to get void installed, we need to do a few preparations of the disks and on the host distro.
### Partitioning
These disk have been partitioned on a different machine some time ago, and I don't have the commands I used recorded, but here is the end result:
```bash
$ lsblk -I 259 -o NAME,KNAME,LABEL,PARTLABEL,FSTYPE,PARTTYPENAME,PARTFLAGS
NAME        KNAME     LABEL PARTLABEL  FSTYPE PARTTYPENAME     PARTFLAGS
nvme1n1     nvme1n1                                            
└─nvme1n1p1 nvme1n1p1 HOME  home       ext4   Linux filesystem 
nvme0n1     nvme0n1                                            
├─nvme0n1p1 nvme0n1p1       grub              BIOS boot        0x1
├─nvme0n1p2 nvme0n1p2 ESP   efi-system vfat   EFI System       
├─nvme0n1p3 nvme0n1p3 BOOT  boot       ext4   Linux filesystem 
├─nvme0n1p4 nvme0n1p4 SWAP  swap       swap   Linux swap       0x8000000000000000
└─nvme0n1p5 nvme0n1p5 ROOT  root       ext4   Linux filesystem 
# parted /dev/nvme0n1 print
Model: Samsung SSD 970 EVO Plus 500GB (nvme)
Disk /dev/nvme0n1: 500GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system     Name        Flags
 1      1049kB  3146kB  2097kB                  grub        hidden, bios_grub
 2      3146kB  137MB   134MB   fat32           efi-system  boot, esp
 3      137MB   1211MB  1074MB  ext4            boot
 4      1211MB  104GB   103GB   linux-swap(v1)  swap        swap
 5      104GB   500GB   396GB   ext4            root

```
The bios_grub partition isn't really necessary, but it is 2 MiB and I can't be bothered to repartition the disk for it. The swap is 1½ times my RAM, allowing for suspend-to-disk (hibernate), which is excessive when I have 64 GiB RAM, but as I don't expect to fully utilise this drive anyway, I gather it will be just fine. The swap has the attribute 63 (do not automount) set using `gdisk`, which shows up as 0x8000000000000000 in `lsblk`.
The intention is for the final disk layout to look like this:
```
/	nvme0n1p5	LUKS2-Argon2i
├─home	nvme1n1p1	LUKS2-Argon2i
└─boot	nvme0n1p3	LUKS2-PBKDF2
  └─efi	nvme0n1p1	
swap	nvme0n1p4	LUKS2-Argon2i
```
### Manjaro 
First of all, we want to update our live usbs packages. As we need `extra/efitools`:
```bash
# pacman -Fy
```
And then all the packages:
```bash
# pacman -Syyu
```
Installing `extra/efitools`:
```bash
# pacman -S extra/efitools
```
And then we need to install `xbps-git` from AUR. For this I used Pamac ("Add/Remove Software") with Preferences:Third Party:AUR:Enable AUR, and then 🔍:AUR:xbps-git.



