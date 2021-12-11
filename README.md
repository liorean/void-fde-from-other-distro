# Installing Void-linux
This is my try to install Void-linux with full disk encryption, specifically the x86_64 version with glibc as system clib, on a Secure Boot system, from a Manjaro KDE live usb because even without a PK key my motherboard firmware (it does not have a separate "disable Secure Boot" option) does not seem to allow booting from the Void live usb. Manjaro is an Arch based distro, and as such provide the very useful AUR package for the Void-linux package handler, XBPS.
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
├─home/	nvme1n1p1	LUKS2-Argon2i
└─boot/	nvme0n1p3	LUKS2-PBKDF2
  └─efi/	nvme0n1p1	
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




https://developer.atmosphereiot.com/documents/hardwareselection/espduino32.html#project-specifics
https://www.gnu.org/software/grub/manual/grub/html_node/Using-digital-signatures.html
https://www.man7.org/linux/man-pages/man8/cryptsetup.8.html

https://unix.stackexchange.com/questions/655214/configure-grub2-to-use-a-keyfile-to-unlock-luks-encrypted-and-boot
https://ruderich.org/simon/notes/secure-boot-with-grub-and-signed-linux-and-initrd

https://medium.com/100-days-of-linux/chroot-a-linux-wonder-fc36ed08087e
https://rlbcontractor.com/fixing-a-broken-linux-system-with-chroot
https://archived.forum.manjaro.org/t/how-to-chroot-into-an-encrypted-root-partition/10760
https://www.preney.ca/paul/archives/389
https://wiki.archlinux.org/title/GRUB
https://wiki.archlinux.org/title/Dm-crypt
https://wiki.archlinux.org/title/Dm-crypt/Swap_encryption
https://wiki.archlinux.org/title/Dm-crypt/Device_encryption
https://wiki.archlinux.org/title/Dm-crypt/Encrypting_a_non-root_file_system
https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system
https://wiki.archlinux.org/title/Data-at-rest_encryption
https://wiki.gentoo.org/wiki/Dm-crypt_full_disk_encryption
https://wiki.gentoo.org/wiki/Full_Disk_Encryption_From_Scratch_Simplified#Automatic_mount_of_encrypted_disk_at_boot
https://wiki.gentoo.org/wiki/User:Sakaki/Sakaki%27s_EFI_Install_Guide/Preparing_the_LUKS-LVM_Filesystem_and_Boot_USB_Key
https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions
https://askubuntu.com/questions/1032546/should-i-use-luks1-or-luks2-for-partition-encryption
https://cryptsetup-team.pages.debian.net/cryptsetup/encrypted-boot.html
https://www.johndstech.com/booting/how-to-grub2-for-uefi-and-luks-encrypted-volumes-for-arch-linux-and-windows-10/
https://wiki.voidlinux.org/Full_Disk_Encryption_w/Encrypted_Boot
https://wiki.voidlinux.org/Manual_Install_with_encrypted_boot
https://wiki.voidlinux.org/Install_alongside_Arch_Linux
https://docs.voidlinux.org/xbps/troubleshooting/static.html
https://docs.voidlinux.org/installation/guides/chroot.html
https://docs.voidlinux.org/installation/guides/fde.html
https://docs.voidlinux.org/xbps/advanced-usage.html
https://aur.archlinux.org/packages/xbps/
https://wiki.archlinux.org/title/Pacman
https://linuxhint.com/create-ramdisk-linux/
