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


For creating encrypted keyfiles, we need GNU's PGP implementation:
Installing `gpg`:
```bash
# pacman -S gnupg
```

```bash
# mkdir -p /distro/{home,boot,root,}
# cryptsetup luksFormat -y -v --type luks2 /dev/nvme1n1p1
# cryptsetup luksOpen /dev/nvme1n1p1 home
# mkfs.ext4 /dev/mapper/home  
# mkdir /distro/home
# ls -Rla distro
```




Home partition:
```
   /mnt/grub    master  lsblk -I 259 -o NAME,KNAME,LABEL,PARTLABEL,FSTYPE,PARTTYPENAME,PARTFLAGS                                                                                                                                ✔ 
NAME        KNAME     LABEL PARTLABEL  FSTYPE PARTTYPENAME     PARTFLAGS
nvme1n1     nvme1n1                                            
`-nvme1n1p1 nvme1n1p1 HOME  home       ext4   Linux filesystem 
nvme0n1     nvme0n1                                            
|-nvme0n1p1 nvme0n1p1       grub              BIOS boot        0x1
|-nvme0n1p2 nvme0n1p2 ESP   efi-system vfat   EFI System       
|-nvme0n1p3 nvme0n1p3 BOOT  boot       ext4   Linux filesystem 
|-nvme0n1p4 nvme0n1p4 SWAP  swap       swap   Linux swap       0x8000000000000000
`-nvme0n1p5 nvme0n1p5 ROOT  root       ext4   Linux filesystem 
    /mnt/grub    master  cryptsetup luksFormat -y -v --type luks2 /dev/nvme1n1p1
Device /dev/nvme1n1p1 does not exist or access denied.
Command failed with code -4 (wrong device or file specified).
    /mnt/grub    master ?6  sudo cryptsetup luksFormat -y -v --type luks2 /dev/nvme1n1p1
WARNING: Device /dev/nvme1n1p1 already contains a 'ext4' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme1n1p1 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme1n1p1: 
Verify passphrase: 
Passphrases do not match.
Command failed with code -2 (no permission or bad passphrase).
    /mnt/grub    master ?6  sudo cryptsetup luksFormat -y -v --type luks2 /dev/nvme1n1p1
WARNING: Device /dev/nvme1n1p1 already contains a 'ext4' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme1n1p1 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme1n1p1: 
Verify passphrase: 
Existing 'ext4' superblock signature on device /dev/nvme1n1p1 will be wiped.
Key slot 0 created.
Command successful.
    /mnt/grub    master ?6  lsblk -I 259 -o NAME,KNAME,LABEL,PARTLABEL,FSTYPE,PARTTYPENAME,PARTFLAGS
NAME KNAME LABEL PARTLABEL FSTYPE PARTTYPENAME PARTFLAGS
nvme1n1
     nvme1n1
                                               
`-nvme1n1p1
     nvme1n1p1
                 home      crypto Linux filesystem
                                               
nvme0n1
     nvme0n1
                                               
|-nvme0n1p1
|    nvme0n1p1
|                grub             BIOS boot    0x1
|-nvme0n1p2
|    nvme0n1p2
|          ESP   efi-system
|                          vfat   EFI System   
|-nvme0n1p3
|    nvme0n1p3
|          BOOT  boot      ext4   Linux filesystem
|                                              
|-nvme0n1p4
|    nvme0n1p4
|          SWAP  swap      swap   Linux swap   0x8000000000000000
`-nvme0n1p5
     nvme0n1p5
           ROOT  root      ext4   Linux filesystem
```

Root partition
```
    ~  sudo cryptsetup luksFormat -y -v --type luks2 /dev/nvme0n1p5                                                                                                           1 ✘  23s  
WARNING: Device /dev/nvme0n1p5 already contains a 'ext4' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme0n1p5 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme0n1p5: 
Verify passphrase: 
Existing 'ext4' superblock signature on device /dev/nvme0n1p5 will be wiped.
Key slot 0 created.
Command successful.
    ~                                                                                                                                                                           ✔  59s  
```
Swap partition
```
sudo cryptsetup luksFormat -y -v --type luks2 /dev/nvme0n1p4                                                                                                           2 ✘  40s  
WARNING: Device /dev/nvme0n1p4 already contains a 'swap' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme0n1p4 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme0n1p4: 
Verify passphrase: 
Existing 'swap' superblock signature on device /dev/nvme0n1p4 will be wiped.
Key slot 0 created.
Command successful.
    ~                                                                                                                                                                           ✔  48s  
```
Boot partition
```
    ~  sudo cryptsetup luksFormat -y -v --type luks2 --pbkdf pbkdf2 /dev/nvme0n1p3                                                                                                  INT ✘ 
WARNING: Device /dev/nvme0n1p3 already contains a 'ext4' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme0n1p3 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme0n1p3: 
Verify passphrase: 
Existing 'ext4' superblock signature on device /dev/nvme0n1p3 will be wiped.
Key slot 0 created.
Command successful.
    ~                                                                                                                                                                           ✔  45s  
```

```
    ~  lsblk -I 259 -o NAME,KNAME,LABEL,PARTLABEL,FSTYPE,PARTTYPENAME,PARTFLAGS                                                                                                 ✔  45s  
NAME        KNAME     LABEL PARTLABEL  FSTYPE      PARTTYPENAME     PARTFLAGS
nvme1n1     nvme1n1                                                 
└─nvme1n1p1 nvme1n1p1       home       crypto_LUKS Linux filesystem 
nvme0n1     nvme0n1                                                 
├─nvme0n1p1 nvme0n1p1       grub                   BIOS boot        0x1
├─nvme0n1p2 nvme0n1p2 ESP   efi-system vfat        EFI System       
├─nvme0n1p3 nvme0n1p3       boot       crypto_LUKS Linux filesystem 
├─nvme0n1p4 nvme0n1p4       swap       crypto_LUKS Linux swap       0x8000000000000000
└─nvme0n1p5 nvme0n1p5       root       crypto_LUKS Linux filesystem 
    ~                                                                                                                                                                                   ✔
```

```
sudo cryptsetup luksOpen /dev/nvme1n1p1 voidhome                 ✔ 
Enter passphrase for /dev/nvme1n1p1: 
    ~  sudo cryptsetup luksOpen /dev/nvme0n1p3 voidboot         ✔  43s  
Enter passphrase for /dev/nvme0n1p3: 
    ~  sudo cryptsetup luksOpen /dev/nvme0n1p4 voidswap         ✔  27s  
Enter passphrase for /dev/nvme0n1p4: 
    ~  sudo cryptsetup luksOpen /dev/nvme0n1p5 voidroot         ✔  22s  
Enter passphrase for /dev/nvme0n1p5: 
    ~  sudo mkdir -p /void/{home,boot,root,}                    ✔  19s  
    ~  sudo mkfs.ext4 -L root /dev/mapper/voidroot                   ✔ 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 96631040 4k blocks and 24158208 inodes
Filesystem UUID: 7761fc3c-8c21-4795-9144-ed8c965a34c6
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done     

    ~  sudo mkfs.ext4 -L boot /dev/mapper/voidboot                      ✔ 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 258048 4k blocks and 64512 inodes
Filesystem UUID: a002d6c1-e43b-45a5-8914-75d1346730d9
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

    ~  sudo mkfs.ext4 -L home /dev/mapper/voidhome                      ✔ 
mke2fs 1.46.5 (30-Dec-2021)
/dev/mapper/voidhome contains a ext4 file system
        last mounted on Wed May 11 13:46:29 2022
Proceed anyway? (y,N) n

    ~  sudo mkswap -L swap /dev/mapper/voidswap                         ✔ 
Setting up swapspace version 1, size = 96 GiB (103062433792 bytes)
LABEL=swap, UUID=e349b37f-5363-46cb-a0fe-db02a39735e0

    ~  sudo mkdir -p /void/{home,boot,root,}                            ✔ 
    ~  sudo mount /dev/mapper/voidboot /void/boot                       ✔ 
    ~  sudo mount /dev/mapper/voidhome /void/home                       ✔ 
    ~  sudo mkdir -p /void/boot/efi                                     ✔ 
    ~  sudo mount /dev/nvme0n1p2 /void/boot/efi                         ✔ 
    ~  for dir in dev proc sys run; do sudo mkdir -p /void/$dir ; sudo mount --rbind /$dir /void/$dir ; sudo mount --make-rslave /void/$dir ; done

    ~  sudo tar xvf {{{dir}}}/void-x86_64-ROOTFS-20221001.tar.xz -C /void
    ~  sudo cp /etc/resolv.conf /void/etc/                              ✔ 
    ~  PS1='(chroot) # ' sudo chroot /void/ /bin/bash
 
(chroot) # xbps-install -Su xbps
[*] Updating repository `https://repo-default.voidlinux.org/current/x86_64-repodata' ...
x86_64-repodata: 1796KB [avg rate: 268KB/s]
Package 'xbps' is up to date.
(chroot) # xbps-install -u

Name             Action    Version           New version            Download size
libcap-ng        update    0.8.2_2           0.8.3_2                12KB 
libcrypto1.1     update    1.1.1q_1          1.1.1s_1               1296KB 
libedit          update    20210910.3.1_1    20221030.3.1_1         105KB 
liblz4           update    1.9.3_1           1.9.4_1                66KB 
liblzma          update    5.2.6_1           5.2.7_1                83KB 
libnftnl         update    1.2.3_1           1.2.4_1                71KB 
libssl1.1        update    1.1.1q_1          1.1.1s_1               228KB 
ncurses          update    6.3_2             6.3_3                  156KB 
ncurses-base     update    6.3_2             6.3_3                  29KB 
ncurses-libs     update    6.3_2             6.3_3                  320KB 
openssh          update    9.0p1_1           9.1p1_2                1076KB 
openssl          update    1.1.1q_1          1.1.1s_1               437KB 
removed-packages update    0.1_73            0.1.20221113_1         5205B 
runit            update    2.1.2_11          2.1.2_12               391KB 
tzdata           update    2022d_1           2022f_2                225KB 
xfsprogs         update    5.18.0_1          5.19.0_1               1160KB 
zlib             update    1.2.12_4          1.2.13_1               53KB 

Size to download:             5722KB
Size required on disk:          22MB
Space available on disk:       362GB

Do you want to continue? [Y/n] y

[*] Downloading packages
libcap-ng-0.8.3_2.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
libcap-ng-0.8.3_2.x86_64.xbps: 12KB [avg rate: 20MB/s]
libcap-ng-0.8.3_2: verifying RSA signature...
libcrypto1.1-1.1.1s_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
libcrypto1.1-1.1.1s_1.x86_64.xbps: 1296KB [avg rate: 301KB/s]
libcrypto1.1-1.1.1s_1: verifying RSA signature...
libedit-20221030.3.1_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
libedit-20221030.3.1_1.x86_64.xbps: 105KB [avg rate: 526KB/s]
libedit-20221030.3.1_1: verifying RSA signature...
liblz4-1.9.4_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
liblz4-1.9.4_1.x86_64.xbps: 66KB [avg rate: 2068KB/s]
liblz4-1.9.4_1: verifying RSA signature...
liblzma-5.2.7_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
liblzma-5.2.7_1.x86_64.xbps: 83KB [avg rate: 1630KB/s]
liblzma-5.2.7_1: verifying RSA signature...
libnftnl-1.2.4_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
libnftnl-1.2.4_1.x86_64.xbps: 71KB [avg rate: 457KB/s]
libnftnl-1.2.4_1: verifying RSA signature...
libssl1.1-1.1.1s_1.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
libssl1.1-1.1.1s_1.x86_64.xbps: 228KB [avg rate: 266KB/s]
libssl1.1-1.1.1s_1: verifying RSA signature...
ncurses-6.3_3.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
ncurses-6.3_3.x86_64.xbps: 156KB [avg rate: 5412KB/s]
ncurses-6.3_3: verifying RSA signature...
ncurses-base-6.3_3.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
ncurses-base-6.3_3.x86_64.xbps: 29KB [avg rate: 799KB/s]
ncurses-base-6.3_3: verifying RSA signature...
ncurses-libs-6.3_3.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
ncurses-libs-6.3_3.x86_64.xbps: 320KB [avg rate: 263KB/s]
ncurses-libs-6.3_3: verifying RSA signature...
openssh-9.1p1_2.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
openssh-9.1p1_2.x86_64.xbps: 1076KB [avg rate: 268KB/s]
openssh-9.1p1_2: verifying RSA signature...
openssl-1.1.1s_1.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
openssl-1.1.1s_1.x86_64.xbps: 437KB [avg rate: 262KB/s]
openssl-1.1.1s_1: verifying RSA signature...
[67%] removed-packages-0.1.20221113_1.x86_64.xbps.sig: [512B 100%] 14MB/s ETA: 0removed-packages-0.1.20221113_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
[67%] removed-packages-0.1.20221113_1.x86_64.xbps: [5205B 78%] 91MB/s ETA: 00m00removed-packages-0.1.20221113_1.x86_64.xbps: 5205B [avg rate: 115MB/s]
removed-packages-0.1.20221113_1: verifying RSA signature...
runit-2.1.2_12.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
runit-2.1.2_12.x86_64.xbps: 391KB [avg rate: 429KB/s]
runit-2.1.2_12: verifying RSA signature...
tzdata-2022f_2.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
tzdata-2022f_2.x86_64.xbps: 225KB [avg rate: 525KB/s]
tzdata-2022f_2: verifying RSA signature...
xfsprogs-5.19.0_1.x86_64.xbps.sig: 512B [avg rate: 26MB/s]
xfsprogs-5.19.0_1.x86_64.xbps: 1160KB [avg rate: 258KB/s]
xfsprogs-5.19.0_1: verifying RSA signature...
zlib-1.2.13_1.x86_64.xbps.sig: 512B [avg rate: 19MB/s]
zlib-1.2.13_1.x86_64.xbps: 53KB [avg rate: 1172KB/s]
zlib-1.2.13_1: verifying RSA signature...

[*] Collecting package files
libcap-ng-0.8.3_2: collecting files...
libcap-ng-0.8.2_2: collecting files...
libcrypto1.1-1.1.1s_1: collecting files...
libcrypto1.1-1.1.1q_1: collecting files...
libedit-20221030.3.1_1: collecting files...
libedit-20210910.3.1_1: collecting files...
liblz4-1.9.4_1: collecting files...
liblz4-1.9.3_1: collecting files...
liblzma-5.2.7_1: collecting files...
liblzma-5.2.6_1: collecting files...
libnftnl-1.2.4_1: collecting files...
libnftnl-1.2.3_1: collecting files...
libssl1.1-1.1.1s_1: collecting files...
libssl1.1-1.1.1q_1: collecting files...
ncurses-6.3_3: collecting files...
ncurses-6.3_2: collecting files...
ncurses-base-6.3_3: collecting files...
ncurses-base-6.3_2: collecting files...
ncurses-libs-6.3_3: collecting files...
ncurses-libs-6.3_2: collecting files...
openssh-9.1p1_2: collecting files...
openssh-9.0p1_1: collecting files...
openssl-1.1.1s_1: collecting files...
openssl-1.1.1q_1: collecting files...
removed-packages-0.1.20221113_1: collecting files...
runit-2.1.2_12: collecting files...
runit-2.1.2_11: collecting files...
tzdata-2022f_2: collecting files...
tzdata-2022d_1: collecting files...
xfsprogs-5.19.0_1: collecting files...
xfsprogs-5.18.0_1: collecting files...
zlib-1.2.13_1: collecting files...
zlib-1.2.12_4: collecting files...

[*] Unpacking packages
libcap-ng-0.8.2_2: updating to 0.8.3_2 ...
libcap-ng-0.8.3_2: unpacking ...
libcrypto1.1-1.1.1q_1: updating to 1.1.1s_1 ...
libcrypto1.1-1.1.1s_1: unpacking ...
libedit-20210910.3.1_1: updating to 20221030.3.1_1 ...
libedit-20221030.3.1_1: unpacking ...
liblz4-1.9.3_1: updating to 1.9.4_1 ...
liblz4-1.9.4_1: unpacking ...
liblzma-5.2.6_1: updating to 5.2.7_1 ...
liblzma-5.2.7_1: unpacking ...
libnftnl-1.2.3_1: updating to 1.2.4_1 ...
libnftnl-1.2.4_1: unpacking ...
libssl1.1-1.1.1q_1: updating to 1.1.1s_1 ...
libssl1.1-1.1.1s_1: unpacking ...
ncurses-6.3_2: updating to 6.3_3 ...
ncurses-6.3_3: unpacking ...
ncurses-base-6.3_2: updating to 6.3_3 ...
ncurses-base-6.3_3: unpacking ...
ncurses-libs-6.3_2: updating to 6.3_3 ...
ncurses-libs-6.3_3: unpacking ...
openssh-9.0p1_1: updating to 9.1p1_2 ...
openssh-9.1p1_2: unpacking ...
Updating configuration file `/etc/ssh/moduli' provided by `openssh-9.1p1_2'.
openssl-1.1.1q_1: updating to 1.1.1s_1 ...
openssl-1.1.1s_1: unpacking ...
removed-packages-0.1_73: updating to 0.1.20221113_1 ...
removed-packages-0.1.20221113_1: unpacking ...
runit-2.1.2_11: updating to 2.1.2_12 ...
runit-2.1.2_12: unpacking ...
tzdata-2022d_1: updating to 2022f_2 ...
tzdata-2022f_2: unpacking ...
xfsprogs-5.18.0_1: updating to 5.19.0_1 ...
xfsprogs-5.19.0_1: unpacking ...
zlib-1.2.12_4: updating to 1.2.13_1 ...
zlib-1.2.13_1: unpacking ...

[*] Configuring unpacked packages
libcap-ng-0.8.3_2: configuring ...
libcap-ng-0.8.3_2: updated successfully.
libcrypto1.1-1.1.1s_1: configuring ...
libcrypto1.1-1.1.1s_1: updated successfully.
libedit-20221030.3.1_1: configuring ...
libedit-20221030.3.1_1: updated successfully.
liblz4-1.9.4_1: configuring ...
liblz4-1.9.4_1: updated successfully.
liblzma-5.2.7_1: configuring ...
liblzma-5.2.7_1: updated successfully.
libnftnl-1.2.4_1: configuring ...
libnftnl-1.2.4_1: updated successfully.
libssl1.1-1.1.1s_1: configuring ...
libssl1.1-1.1.1s_1: updated successfully.
ncurses-6.3_3: configuring ...
ncurses-6.3_3: updated successfully.
ncurses-base-6.3_3: configuring ...
ncurses-base-6.3_3: updated successfully.
ncurses-libs-6.3_3: configuring ...
ncurses-libs-6.3_3: updated successfully.
openssh-9.1p1_2: configuring ...
openssh-9.1p1_2: updated successfully.
openssl-1.1.1s_1: configuring ...
openssl-1.1.1s_1: updated successfully.
removed-packages-0.1.20221113_1: configuring ...
removed-packages-0.1.20221113_1: updated successfully.
runit-2.1.2_12: configuring ...
runit-2.1.2_12: updated successfully.
tzdata-2022f_2: configuring ...
tzdata-2022f_2: updated successfully.
xfsprogs-5.19.0_1: configuring ...
xfsprogs-5.19.0_1: updated successfully.
zlib-1.2.13_1: configuring ...
zlib-1.2.13_1: updated successfully.

17 downloaded, 0 installed, 17 updated, 17 configured, 0 removed.
(chroot) # xbps-install base-system
25 packages will be downloaded:

25 packages will be installed:
  libusb-1.0.26_1 usbutils-014_2 dbus-libs-1.14.4_1 
  wpa_supplicant-2.10_1 ipw2100-firmware-1.3_6 ipw2200-firmware-3.1_6 
  zd1211-firmware-1.5_3 wifi-firmware-1.3_4 void-artwork-20221013_1 
  ethtool-5.19_1 acpid-2.0.33_2 linux-firmware-amd-20220411_1 
  linux-firmware-intel-20220411_1 linux-firmware-nvidia-20220411_1 
  linux-firmware-broadcom-20220411_1 linux-firmware-network-20220411_1 
  cpio-2.13_1 libaio-0.3.112_1 device-mapper-2.02.187_2 
  kpartx-0.9.3_1 dracut-056_1 linux-base-2021.07.21_1 
  linux6.0-6.0.9_1 linux-6.0_1 base-system-0.114_1 

Size to download:              306MB
Size required on disk:         670MB
Space available on disk:       362GB

Do you want to continue? [Y/n] y

[*] Downloading packages
libusb-1.0.26_1.x86_64.xbps.sig: 512B [avg rate: 22MB/s]
libusb-1.0.26_1.x86_64.xbps: 53KB [avg rate: 113MB/s]
libusb-1.0.26_1: verifying RSA signature...
usbutils-014_2.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
usbutils-014_2.x86_64.xbps: 84KB [avg rate: 1499KB/s]
usbutils-014_2: verifying RSA signature...
dbus-libs-1.14.4_1.x86_64.xbps.sig: 512B [avg rate: 22MB/s]
dbus-libs-1.14.4_1.x86_64.xbps: 149KB [avg rate: 1429KB/s]
dbus-libs-1.14.4_1: verifying RSA signature...
wpa_supplicant-2.10_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
wpa_supplicant-2.10_1.x86_64.xbps: 1301KB [avg rate: 289KB/s]
wpa_supplicant-2.10_1: verifying RSA signature...
ipw2100-firmware-1.3_6.x86_64.xbps.sig: 512B [avg rate: 7813KB/s]
ipw2100-firmware-1.3_6.x86_64.xbps: 174KB [avg rate: 368KB/s]
ipw2100-firmware-1.3_6: verifying RSA signature...
ipw2200-firmware-3.1_6.x86_64.xbps.sig: 512B [avg rate: 23MB/s]
ipw2200-firmware-3.1_6.x86_64.xbps: 147KB [avg rate: 2215KB/s]
ipw2200-firmware-3.1_6: verifying RSA signature...
zd1211-firmware-1.5_3.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
zd1211-firmware-1.5_3.x86_64.xbps: 12KB [avg rate: 14MB/s]
zd1211-firmware-1.5_3: verifying RSA signature...
wifi-firmware-1.3_4.x86_64.xbps.sig: 512B [avg rate: 22MB/s]
wifi-firmware-1.3_4.x86_64.xbps: 544B [avg rate: 22MB/s]
wifi-firmware-1.3_4: verifying RSA signature...
void-artwork-20221013_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
void-artwork-20221013_1.x86_64.xbps: 254KB [avg rate: 6544KB/s]
void-artwork-20221013_1: verifying RSA signature...
ethtool-5.19_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
ethtool-5.19_1.x86_64.xbps: 212KB [avg rate: 4180KB/s]
ethtool-5.19_1: verifying RSA signature...
acpid-2.0.33_2.x86_64.xbps.sig: 512B [avg rate: 22MB/s]
acpid-2.0.33_2.x86_64.xbps: 55KB [avg rate: 328KB/s]
acpid-2.0.33_2: verifying RSA signature...
[ 0%] linux-firmware-amd-20220411_1.x86_64.xbps.sig: [512B 100%] 21MB/s ETA: 00mlinux-firmware-amd-20220411_1.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
linux-firmware-amd-20220411_1.x86_64.xbps: 13MB [avg rate: 258KB/s]
linux-firmware-amd-20220411_1: verifying RSA signature...
[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps.sig: [512B 100%] 20MB/s ETA: 0linux-firmware-intel-20220411_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 0%] 266KB/s ETA: 00m0[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 1%] 231KB/s ETA: 01m0[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 9%] 250KB/s ETA: 00m1[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 17%] 254KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 23%] 237KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 32%] 255KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 40%] 254KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 48%] 254KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 55%] 255KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 63%] 255KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 71%] 254KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 78%] 254KB/s ETA: 00m[ 5%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 86%] 255KB/s ETA: 00m[ 6%] linux-firmware-intel-20220411_1.x86_64.xbps: [3306KB 94%] 255KB/s ETA: 00mlinux-firmware-intel-20220411_1.x86_64.xbps: 3306KB [avg rate: 271KB/s]
linux-firmware-intel-20220411_1: verifying RSA signature...
[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps.sig: [512B 100%] 21MB/s ETA: linux-firmware-nvidia-20220411_1.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 0%] 79KB/s ETA: 00m0[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 4%] 243KB/s ETA: 00m[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 27%] 252KB/s ETA: 00[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 50%] 253KB/s ETA: 00[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 73%] 255KB/s ETA: 00[ 6%] linux-firmware-nvidia-20220411_1.x86_64.xbps: [1112KB 96%] 254KB/s ETA: 00linux-firmware-nvidia-20220411_1.x86_64.xbps: 1112KB [avg rate: 263KB/s]
linux-firmware-nvidia-20220411_1: verifying RSA signature...
[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps.sig: [512B 100%] 14MB/s ETAlinux-firmware-broadcom-20220411_1.x86_64.xbps.sig: 512B [avg rate: 14MB/s]
[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 0%] 153KB/s ETA: 0[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 2%] 250KB/s ETA: 0[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 6%] 228KB/s ETA: 0[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 9%] 248KB/s ETA: 0[ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 13%] 251KB/s ETA: [ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 16%] 251KB/s ETA: [ 6%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 20%] 251KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 23%] 251KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 27%] 252KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 30%] 252KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 34%] 252KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 37%] 248KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 41%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 44%] 249KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 48%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 51%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 55%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 58%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 62%] 253KB/s ETA: [ 7%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 65%] 253KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 68%] 251KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 72%] 254KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 76%] 251KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 79%] 253KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 83%] 254KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 86%] 253KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 90%] 254KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 93%] 254KB/s ETA: [ 8%] linux-firmware-broadcom-20220411_1.x86_64.xbps: [7264KB 97%] 254KB/s ETA: linux-firmware-broadcom-20220411_1.x86_64.xbps: 7264KB [avg rate: 261KB/s]
linux-firmware-broadcom-20220411_1: verifying RSA signature...
[ 8%] linux-firmware-network-20220411_1.x86_64.xbps.sig: [512B 100%] 21MB/s ETA:linux-firmware-network-20220411_1.x86_64.xbps.sig: 512B [avg rate: 21MB/s]
[ 8%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 121KB/s ETA: 00m[ 8%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 237KB/s ETA: 36m[ 8%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 248KB/s ETA: 13m[ 8%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 251KB/s ETA: 10m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 251KB/s ETA: 09m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 0%] 252KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 1%] 252KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 1%] 254KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 1%] 248KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 1%] 254KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 2%] 253KB/s ETA: 08m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 2%] 254KB/s ETA: 07m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 2%] 254KB/s ETA: 07m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 2%] 253KB/s ETA: 07m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 2%] 254KB/s ETA: 07m[ 9%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 3%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 3%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 3%] 251KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 3%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 4%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 4%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 4%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 4%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 4%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 5%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 5%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 5%] 252KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 5%] 254KB/s ETA: 07m[10%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 6%] 252KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 6%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 6%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 6%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 6%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 7%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 7%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 7%] 253KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 7%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 8%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 8%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 8%] 254KB/s ETA: 07m[11%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 8%] 254KB/s ETA: 07m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 8%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 9%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 9%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 9%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 9%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 9%] 254KB/s ETA: 06m[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 10%] 254KB/s ETA: 06[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 10%] 254KB/s ETA: 06[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 10%] 254KB/s ETA: 06[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 10%] 254KB/s ETA: 06[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 11%] 254KB/s ETA: 06[12%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 11%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 11%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 11%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 11%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 12%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 12%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 12%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 12%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 13%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 13%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 13%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 13%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 13%] 254KB/s ETA: 06[13%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 14%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 14%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 14%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 14%] 253KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 15%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 15%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 15%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 15%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 15%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 16%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 16%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 16%] 254KB/s ETA: 06[14%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 16%] 253KB/s ETA: 06
[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 17%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 17%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 17%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 17%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 17%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 18%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 18%] 253KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 18%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 18%] 253KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 19%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 19%] 254KB/s ETA: 06[15%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 19%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 19%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 19%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 20%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 20%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 20%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 20%] 254KB/s ETA: 06[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 21%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 21%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 21%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 21%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 21%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 22%] 254KB/s ETA: 05[16%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 22%] 253KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 22%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 22%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 23%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 23%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 23%] 253KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 23%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 23%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 24%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 24%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 24%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 24%] 254KB/s ETA: 05[17%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 25%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 25%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 25%] 253KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 25%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 25%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 26%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 26%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 26%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 26%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 26%] 253KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 27%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 27%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 27%] 254KB/s ETA: 05[18%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 27%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 28%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 28%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 28%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 28%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 29%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 29%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 29%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 29%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 29%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 30%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 30%] 254KB/s ETA: 05[19%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 30%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 30%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 31%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 31%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 31%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 31%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 31%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 32%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 32%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 32%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 32%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 33%] 254KB/s ETA: 05[20%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 33%] 254KB/s ETA: 05[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 33%] 254KB/s ETA: 05[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 33%] 254KB/s ETA: 05[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 33%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 34%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 34%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 34%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 34%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 35%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 35%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 35%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 35%] 254KB/s ETA: 04[21%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 35%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 36%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 36%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 36%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 36%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 37%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 37%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 37%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 37%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 37%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 38%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 38%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 38%] 254KB/s ETA: 04[22%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 38%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 39%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 39%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 39%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 39%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 39%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 40%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 40%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 40%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 40%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 41%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 41%] 254KB/s ETA: 04[23%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 41%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 41%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 41%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 42%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 42%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 42%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 42%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 43%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 43%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 43%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 43%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 43%] 254KB/s ETA: 04[24%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 44%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 44%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 44%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 44%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 45%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 45%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 45%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 45%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 45%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 46%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 46%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 46%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 46%] 254KB/s ETA: 04[25%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 47%] 254KB/s ETA: 04[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 47%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 47%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 47%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 47%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 48%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 48%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 48%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 48%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 49%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 49%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 49%] 254KB/s ETA: 03[26%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 49%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 49%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 50%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 50%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 50%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 50%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 50%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 51%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 51%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 51%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 51%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 52%] 254KB/s ETA: 03[27%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 52%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 52%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 52%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 52%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 53%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 53%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 53%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 53%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 54%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 54%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 54%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 54%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 54%] 254KB/s ETA: 03[28%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 55%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 55%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 55%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 55%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 56%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 56%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 56%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 56%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 56%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 57%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 57%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 57%] 254KB/s ETA: 03[29%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 57%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 58%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 58%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 58%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 58%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 58%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 59%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 59%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 59%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 59%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 60%] 254KB/s ETA: 03[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 60%] 254KB/s ETA: 02[30%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 60%] 254KB/s ETA: 02
[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 60%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 60%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 61%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 61%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 61%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 61%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 62%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 62%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 62%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 62%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 62%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 63%] 254KB/s ETA: 02[31%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 63%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 63%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 63%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 64%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 64%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 64%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 64%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 64%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 65%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 65%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 65%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 65%] 254KB/s ETA: 02[32%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 66%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 66%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 66%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 66%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 66%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 67%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 67%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 67%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 67%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 68%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 68%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 68%] 254KB/s ETA: 02[33%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 68%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 68%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 69%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 69%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 69%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 69%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 70%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 70%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 70%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 70%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 70%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 71%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 71%] 254KB/s ETA: 02[34%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 71%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 71%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 72%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 72%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 72%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 72%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 72%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 73%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 73%] 254KB/s ETA: 02[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 73%] 254KB/s ETA: 01[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 73%] 254KB/s ETA: 01[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 74%] 254KB/s ETA: 01[35%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 74%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 74%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 74%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 74%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 75%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 75%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 75%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 75%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 76%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 76%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 76%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 76%] 254KB/s ETA: 01[36%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 76%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 77%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 77%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 77%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 77%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 78%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 78%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 78%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 78%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 78%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 79%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 79%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 79%] 254KB/s ETA: 01[37%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 79%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 80%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 80%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 80%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 80%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 80%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 81%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 81%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 81%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 81%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 82%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 82%] 254KB/s ETA: 01[38%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 82%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 82%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 82%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 83%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 83%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 83%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 83%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 84%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 84%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 84%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 84%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 84%] 254KB/s ETA: 01[39%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 85%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 85%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 85%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 85%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 86%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 86%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 86%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 86%] 254KB/s ETA: 01[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 86%] 254KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 254KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 254KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 254KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 253KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 253KB/s ETA: 00[40%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 87%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 88%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 88%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 88%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 88%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 89%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 89%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 89%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 89%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 89%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 90%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 90%] 253KB/s ETA: 00[41%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 90%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 90%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 91%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 91%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 91%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 91%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 91%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 253KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 252KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 252KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 252KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 92%] 252KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 93%] 252KB/s ETA: 00[42%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 93%] 252KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 93%] 252KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 93%] 252KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 252KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 252KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 94%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 95%] 251KB/s ETA: 00[43%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 96%] 251KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 96%] 251KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 96%] 251KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 96%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 96%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 97%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 97%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 97%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 97%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 97%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 250KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 249KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 249KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 249KB/s ETA: 00[44%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 98%] 249KB/s ETA: 00[45%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 99%] 249KB/s ETA: 00[45%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 99%] 249KB/s ETA: 00[45%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 99%] 249KB/s ETA: 00[45%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 99%] 249KB/s ETA: 00[45%] linux-firmware-network-20220411_1.x86_64.xbps: [112MB 99%] 249KB/s ETA: 00
linux-firmware-network-20220411_1.x86_64.xbps: 112MB [avg rate: 249KB/s]
linux-firmware-network-20220411_1: verifying RSA signature...
cpio-2.13_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
cpio-2.13_1.x86_64.xbps: 196KB [avg rate: 9.9MB/s]
cpio-2.13_1: verifying RSA signature...
libaio-0.3.112_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
libaio-0.3.112_1.x86_64.xbps: 2732B [avg rate: 79MB/s]
libaio-0.3.112_1: verifying RSA signature...
device-mapper-2.02.187_2.x86_64.xbps.sig: 512B [avg rate: 26MB/s]
device-mapper-2.02.187_2.x86_64.xbps: 1028KB [avg rate: 245KB/s]
device-mapper-2.02.187_2: verifying RSA signature...
kpartx-0.9.3_1.x86_64.xbps.sig: 512B [avg rate: 13MB/s]
kpartx-0.9.3_1.x86_64.xbps: 26KB [avg rate: 370KB/s]
kpartx-0.9.3_1: verifying RSA signature...
dracut-056_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
dracut-056_1.x86_64.xbps: 235KB [avg rate: 168KB/s]
dracut-056_1: verifying RSA signature...
linux-base-2021.07.21_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
linux-base-2021.07.21_1.x86_64.xbps: 593B [avg rate: 25MB/s]
linux-base-2021.07.21_1: verifying RSA signature...
linux6.0-6.0.9_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
linux6.0-6.0.9_1.x86_64.xbps: 166MB [avg rate: 204KB/s]
linux6.0-6.0.9_1: verifying RSA signature...
linux-6.0_1.x86_64.xbps.sig: 512B [avg rate: 20MB/s]
linux-6.0_1.x86_64.xbps: 539B [avg rate: 22MB/s]
linux-6.0_1: verifying RSA signature...
base-system-0.114_1.x86_64.xbps.sig: 512B [avg rate: 22MB/s]
base-system-0.114_1.x86_64.xbps: 846B [avg rate: 31MB/s]
base-system-0.114_1: verifying RSA signature...

[*] Collecting package files
libusb-1.0.26_1: collecting files...
usbutils-014_2: collecting files...
dbus-libs-1.14.4_1: collecting files...
wpa_supplicant-2.10_1: collecting files...
ipw2100-firmware-1.3_6: collecting files...
ipw2200-firmware-3.1_6: collecting files...
zd1211-firmware-1.5_3: collecting files...
wifi-firmware-1.3_4: collecting files...
void-artwork-20221013_1: collecting files...
ethtool-5.19_1: collecting files...
acpid-2.0.33_2: collecting files...
linux-firmware-amd-20220411_1: collecting files...
linux-firmware-intel-20220411_1: collecting files...
linux-firmware-nvidia-20220411_1: collecting files...
linux-firmware-broadcom-20220411_1: collecting files...
linux-firmware-network-20220411_1: collecting files...
cpio-2.13_1: collecting files...
libaio-0.3.112_1: collecting files...
device-mapper-2.02.187_2: collecting files...
kpartx-0.9.3_1: collecting files...
dracut-056_1: collecting files...
linux-base-2021.07.21_1: collecting files...
linux6.0-6.0.9_1: collecting files...
linux-6.0_1: collecting files...
base-system-0.114_1: collecting files...

[*] Unpacking packages
libusb-1.0.26_1: unpacking ...
usbutils-014_2: unpacking ...
dbus-libs-1.14.4_1: unpacking ...
wpa_supplicant-2.10_1: unpacking ...
ipw2100-firmware-1.3_6: unpacking ...
ipw2200-firmware-3.1_6: unpacking ...
zd1211-firmware-1.5_3: unpacking ...
wifi-firmware-1.3_4: unpacking ...
void-artwork-20221013_1: unpacking ...
ethtool-5.19_1: unpacking ...
acpid-2.0.33_2: unpacking ...
linux-firmware-amd-20220411_1: unpacking ...
linux-firmware-intel-20220411_1: unpacking ...
linux-firmware-nvidia-20220411_1: unpacking ...
linux-firmware-broadcom-20220411_1: unpacking ...
linux-firmware-network-20220411_1: unpacking ...
cpio-2.13_1: unpacking ...
libaio-0.3.112_1: unpacking ...
device-mapper-2.02.187_2: unpacking ...
kpartx-0.9.3_1: unpacking ...
dracut-056_1: unpacking ...
dracut-056_1: registered 'initramfs' alternatives group
Creating 'initramfs' alternatives group symlink: /etc/kernel.d/post-install/20-initramfs -> /usr/libexec/dracut/kernel-hook-postinst
Creating 'initramfs' alternatives group symlink: /etc/kernel.d/post-remove/20-initramfs -> /usr/libexec/dracut/kernel-hook-postrm
linux-base-2021.07.21_1: unpacking ...
linux6.0-6.0.9_1: unpacking ...
linux-6.0_1: unpacking ...
base-system-0.114_1: unpacking ...

[*] Configuring unpacked packages
libusb-1.0.26_1: configuring ...
libusb-1.0.26_1: installed successfully.
usbutils-014_2: configuring ...
usbutils-014_2: installed successfully.
dbus-libs-1.14.4_1: configuring ...
dbus-libs-1.14.4_1: installed successfully.
wpa_supplicant-2.10_1: configuring ...
wpa_supplicant-2.10_1: installed successfully.
ipw2100-firmware-1.3_6: configuring ...
ipw2100-firmware-1.3_6: installed successfully.
ipw2200-firmware-3.1_6: configuring ...
ipw2200-firmware-3.1_6: installed successfully.
zd1211-firmware-1.5_3: configuring ...
zd1211-firmware-1.5_3: installed successfully.
wifi-firmware-1.3_4: configuring ...
wifi-firmware-1.3_4: installed successfully.
void-artwork-20221013_1: configuring ...
void-artwork-20221013_1: installed successfully.
ethtool-5.19_1: configuring ...
ethtool-5.19_1: installed successfully.
acpid-2.0.33_2: configuring ...
acpid-2.0.33_2: installed successfully.
linux-firmware-amd-20220411_1: configuring ...
linux-firmware-amd-20220411_1: installed successfully.
linux-firmware-intel-20220411_1: configuring ...
linux-firmware-intel-20220411_1: installed successfully.
linux-firmware-nvidia-20220411_1: configuring ...
linux-firmware-nvidia-20220411_1: installed successfully.
linux-firmware-broadcom-20220411_1: configuring ...
linux-firmware-broadcom-20220411_1: installed successfully.
linux-firmware-network-20220411_1: configuring ...
linux-firmware-network-20220411_1: installed successfully.
cpio-2.13_1: configuring ...
cpio-2.13_1: installed successfully.
libaio-0.3.112_1: configuring ...
libaio-0.3.112_1: installed successfully.
device-mapper-2.02.187_2: configuring ...
device-mapper-2.02.187_2: installed successfully.
kpartx-0.9.3_1: configuring ...
kpartx-0.9.3_1: installed successfully.
dracut-056_1: configuring ...
dracut-056_1: installed successfully.
linux-base-2021.07.21_1: configuring ...
linux-base-2021.07.21_1: installed successfully.
linux6.0-6.0.9_1: configuring ...
Executing post-install kernel hook: 20-initramfs ...
linux6.0-6.0.9_1: installed successfully.
linux-6.0_1: configuring ...
linux-6.0_1: installed successfully.
base-system-0.114_1: configuring ...
base-system-0.114_1: installed successfully.

25 downloaded, 25 installed, 0 updated, 25 configured, 0 removed.
(chroot) # xbps-remove base-voidstrap

Name           Action    Version           New version            Download size
base-voidstrap remove    0.11_1            -                      - 

Space available on disk:       361GB

Do you want to continue? [Y/n] y
Removing `base-voidstrap-0.11_1' ...
Removed `base-voidstrap-0.11_1' successfully.

0 downloaded, 0 installed, 0 updated, 0 configured, 1 removed.
```

```
(chroot) # nvi /etc/hostname
(chroot) # nvi /etc/rc.conf
(chroot) # nvi /etc/default/libc-locales
(chroot) # xbps-reconfigure -f glibc-locales
glibc-locales: configuring ...
Generating GNU libc locales...
  en_GB.UTF-8... done.
  en_US.UTF-8... done.
  sv_SE.UTF-8... done.
  sv_SE.ISO-8859-1... done.
glibc-locales: configured successfully.
(chroot) # passwd
New password: 
Retype new password: 
passwd: password updated successfully

```

/etc/fstab
```
#<file system> <dir> <type> <options> <dump> <pass>
/dev/mapper/voidroot / ext4 rw,noatime 0 0
/dev/mapper/voidboot /boot ext4 rw,noatime 0 0
/dev/mapper/voidhome /home ext4 rw,noatime 0 0
UUID=CC0D-9AAC /boot/efi vfat rw,noatime,fmask=0022,dmask=0022,codepage=437,ioch
arset=ascii,shortname=mixed,utf8,errors=remount-ro 0 0
tmpfs /tmp tmpfs default,nosuid,nodev, 0 0
/dev/mapper/voidswap swap swap rw,noatime,discard 0 0 
```


https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/

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
