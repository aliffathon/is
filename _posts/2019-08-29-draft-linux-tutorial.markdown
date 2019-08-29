
http://kambing.ui.ac.id/iso/ubuntu/releases/16.04/

http://kambing.ui.ac.id/iso/ubuntu/releases/16.04/ubuntu-16.04.6-server-amd64.iso

--------------------

https://lira.epac.to/MyPublicWiki/VBox%20How%20to%20use%20a%20raw%20disk%20image%20file%20in%20VirtualBox.html
https://superuser.com/questions/332252/how-to-create-and-format-a-partition-using-a-bash-script/1132834

dd if=/dev/zero of=disk.img bs=1K count=5M
cfdisk disk.img
dd if=/dev/zero of=disk.img bs=100M count=5

losetup /dev/loop0 /media/alpha/DATA2/disk1.img
fdisk /dev/loop0
o # clear partition table in memory
n # new partition
p # primary
1 # partition number 1
  # default start beginning
+100M # boot partition
n # new partition
p # primary
2 # number
  # start
  # end, all
a # active, bootable
1 # boot partition
p # print in-memory partition table
w # write changes
q # quit

~~~
sed -e 's/\s*\([\+0-9a-zA-Z*]\).*/\1/' << EOF | fdisk /dev/loop0
o
n
p
1
 #empty
+100M
n
p
2
 # empty
 # empty
a
1
p
w
q
~~~

or cloning partition table
sfdisk -d /dev/sda > sda.sfdisk
sfdisk /dev/sdb < sda.sfdisk

( echo o; echo n; echo p; echo 1; echo ""; echo ""; echo w ) | sudo fdisk /dev/loop0

echo -e "o\nn\np\n1\n\n\nw" | fdisk /dev/loop0

fdisk /dev/loop0 <<EOF
n
p
1


w
EOF

fdisk /dev/loop0 < /tmp/fdisk.cmds

/sbin/parted /dev/loop0
 mklabel gpt --script
 mkpart primary 0% 100% --script



1. losetup /dev/loop0 /home/disk.img
2. VBoxManage internalcommands createrawvmdk -filnename /home/disk.img -rawdisk /dev/loop0 -register

1. VBoxManage internalcommands convertfromraw /home/ubuntu.img /home/ubuntu.vdi --format vdi
2. VBoxManage internalcommands converttoraw /home/ubuntu.vdi /home/ubuntu.img


----------------------

http://acahya.web.id/repository-ubuntu-18-04-lts-lokal-iix/

~~~
deb http://kartolo.sby.datautama.net.id/ubuntu/ bionic main restricted
deb http://kebo.pens.ac.id/ubuntu/ bionic main restricted
deb http://mirror.cloud.id/ubuntu/ bionic main restricted
~~~

~~~
deb http://buaya.klas.or.id/ubuntu/ bionic main restricted universe multiverse
deb http://buaya.klas.or.id/ubuntu/ bionic-updates main restricted universe multiverse
deb http://buaya.klas.or.id/ubuntu/ bionic-backports main restricted universe multiverse
deb http://buaya.klas.or.id/ubuntu/ bionic-security main restricted universe multiverse
~~~

~~~
deb http://kambing.ui.ac.id/ubuntu/ xenial main restricted universe multiverse
deb http://kambing.ui.ac.id/ubuntu/ xenial-updates main restricted universe multiverse
deb http://kambing.ui.ac.id/ubuntu/ xenial-backports main restricted universe multiverse
deb http://kambing.ui.ac.id/ubuntu/ xenial-security main restricted universe multiverse
~~~

-----------------

https://www.tecmint.com/useful-basic-commands-of-apt-get-and-apt-cache-for-package-management/
https://www.thegeekstuff.com/2009/10/debian-ubuntu-install-upgrade-remove-packages-using-apt-get-apt-cache-apt-file-dpkg/


1. apt-file search apache2.conf
1. apt-file list apache2 
1. dpkg -l | grep -i apache
1. apt-cache depends apache2
1. apt-cache pkgnames
1. apt-cache pkgnames vsftpd
   apt-cache search vsftpd
   apt-cache search ^apache2$
   apt-cache search "Apache HTTP Server"
1. apt-cache show netcat
1. apt-cache showpkg vsftpd
1. apt-cache stats
1. apt-get update
1. apt-get dist-upgrade
   apt-get upgrade
1. apt-get install netcat
1. apt-get install nethogs goaccess
1. apt-get install '*name*'
1. apt-get install packageName --reinstall
1. apt-get install packageName --no-upgrade
1. apt-get install packageName --only-upgrade
1. apt-get install vsftpd=2.3.5-3ubuntu1
1. apt-get remove vsftpd
1. apt-get remove --purge vsftpd 
   apt-get purge vsftpd
1. apt-get clean
1. apt-get --download-only source vsftpd
1. apt-get source vsftpd
1. apt-get --compile source goaccess
1. apt-get download nethogs
1. apt-get changelog vsftpd
1. apt-get check
1. apt-get build-dep netcat
1. apt-get autoclean
1. apt-get autoremove vsftpd

1. apt-get install
-f --fix-broken 					APT::Get::Fix-Broken
-d --download-only 					APT::Get::Download-Only
--install-suggests 					APT::Install-Suggests
--no-install-recommends 			APT::Install-Recommends
-m --ignore-missing --fix-missing 	APT::Get::Fix-Missing
--no-download 						APT::Get::Download 		via deb di archive
-q --quiet -qq
-s --simulate --just-print --dry-run --recon --no-act
-y --yes --assume-yes APT::Get::Assume-Yes
--assume-no APT::Get::Assume-No
-u --show-upgraded APT::Get::Show-Upgraded
-V --verbose-versions APT::Get::Show-Versions
-a --host-architecture APT::Get::Host-Architecture
-P --build-profiles APT::Build-Profiles
-b --compile --build APT::Get::Compile
--ignore-hold APT::Ignore-Hold
--with-new-pkgs APT::Get::Upgrade-Allow-New
--no-upgrade APT::Get::Upgrade
--only-upgrade APT::Get:Only-Upgrade
--allow-downgrades APT::Get::allow-downgrades
--force-yes APT::Get::force-yes
--print-uris APT::Get::Print-URIs
--purge APT::Get::Purge
--reinstall APT::Get::ReInstall
--auto-remove --autoremove APT::Get::AutomaticRemove
-c --config-file

files:
Dir::Etc::SourceList /etc/apt/sources.list;
Dir::Etc::SourceParts /etc/apt/sources.list.d/;
Dir::Etc::Main /etc/apt/apt.conf;
Dir::Etc::Parts /etc/apt/apt.conf.d/;
Dir::Etc::Preferences /etc/apt/preferences;
Dir::Etc::Parts /etc/apt/preferences.d/;
Dir::Cache::Archives /var/cache/archives/; 	[partial]
Dir::State::Lists /var/lib/apt/lists/;		[partial]

apt-get -c apt.conf install -d nginx
apt-get -c apt.conf install --no-download nginx


------------------------------

https://lira.epac.to/MyPublicWiki/Clone%20a%20Linux%20system%20install%20to%20another%20computer.html

Clone a Linux system install to another computer



The following method should work for any Linux distribution. Source and target systems must be on the same processor architecture (though transfer from 32bit to 64bit should work).

What you need:

- 2 live USB keys (or cds)
- Good quality ethernet cables (one cable between the 2 computers is OK), or a usb key/drive with a BIG ext4 partition. 


1. Boot source and target machines on live USB/CD

Any live USB/CD should be OK.
On the target computer, you will need a tool to partition your hard drive, like gparted.
rsync is also required for data transfer: it’s included in many live systems.

Ubuntu live cd is OK, Manjaro live cd too.


2. Partition your target hard drive

Use a tool like gparted to partition the target hard drive, with the same partitions as your source system (slash, swap, home…).
I recommend you to assign LABELs to your partitions: for the fstab, it’s easier than UUIDs.


3. Mount all partitions on both machines

On both systems, open a root terminal. Then, for each data partition (you can ignore swap):

mkdir /mnt/slash
mount /dev/sdaX /mnt/slash

If you have a home partition:

mkdir /mnt/home
mount /dev/sdaY /mnt/home


4. Transfer the data (network or usb)

This part may be tricky. Choose the method you prefer.

Network

    Setup the network. Test the connectivity with ping command.
    The easier is to plug the PCs on a DHCP network (like your ISP box) so that you get automatic IP addresses. If you linked the 2 pcs with a single cable, you’ll have to setup the IPs with NetworkManager (static ips, or adhoc network).
    On source system, as root, create a simple /etc/rsyncd.conf file:

    uid = root
    gid = root
    use chroot = no

    [all]
        path = /

    Then start the rsync daemon server: rsync --daemon
    On target PC, for each partition:

    rsync -avHX SOURCE_IP::all/mnt/slash/ /mnt/slash/

    Don’t forget ‘/’ at the end of paths. -a will preserve many file attributes like owner and permissions, -H will preserve hardlinks if any, -X will preserve extended attributes like setuid. You may also add -A if you are using acls. What is good with rsync is that you can stop and restart the transfer whenever you want.


USB

Prepare a USB drive with a BIG ext4 partition.

    Mount the USB partition on source system (mount /dev/sdbX /mnt/usb)
    For each partition:

    rsync -avHX /mnt/slash/ /mnt/usb/slash/

    umount, unplug and remount the USB disk on the target system.
    For each partition:

    rsync -avHX /mnt/usb/slash/ /mnt/slash/


5. Change fstab on target system

As root, edit /mnt/slash/etc/fstab
For each partition (including swap), replace the first field with the new UUID or LABEL (it’s straightforward with LABELs):
UUID=the-long-uuid, or LABEL=yourlabel

2 ways to get the UUIDs / LABELs:

ls -l /dev/disk/by-uuid/
blkid /dev/sdaX


6. Reinstall Grub

We will use a chroot (changed root environment) to be able to call the grub install inside the migrated system.

First, bind mount some system directories needed by grub, then chroot:

mount --bind /proc /mnt/slash/proc
mount --bind /sys /mnt/slash/sys
mount --bind /dev /mnt/slash/dev
mount --bind /run /mnt/slash/run
chroot /mnt/slash

Then install grub in the Master Boot Record of your hard drive, and update grub config file (with the new uuids…):

grub-install /dev/sda
update-grub


7. Reboot target machine

That’s it! Your system should be working on the new computer now.
Feel free to comment if you encounter problems.


-----------------------------------------------


Frank
2014-08-10 at 14:05

Based on your guide, I cloned an actively running Arch system (X server logged off) successfully the following way:

->A primary partition has been created on the new SSD using fdisk, then it has been formatted with EXT4 using mkfs.ext4 and mounted to /mnt/new

->The partition has been cloned using rsync, but /mnt has to be exluded, otherwise it will be copied recursively until the new disc is full:
rsync -avHX –exclude ‘mnt’ / /mnt/new/

->arch-chroot has been run directly at the /mnt/new without the blind mounts.

->grub-mkconfig -o /boot/grub/grub.cfg
->grub-install /dev/sda
->mkinitcpio -p linux#

Reboot…works!

I forgot: the new uuid of the SSD partition has also been edited in the fstab.

--------------

https://lira.epac.to/MyPublicWiki/Bash:%20Validate%20IP%20Addresses.html

Bash: Validate IP Addresses
function validateIP()
 {
         local ip=$1
         local stat=1
         if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
                OIFS=$IFS
                IFS='.'
                ip=($ip)
                IFS=$OIFS
                [[ ${ip[0]} -le 255 && ${ip[1]} -le 255 \
                && ${ip[2]} -le 255 && ${ip[3]} -le 255 ]]
                stat=$?
        fi
        return $stat
}

# ---------------------------------------------
# TEST
# ---------------------------------------------

echo "Enter IP Address"
read ip
validateIP $ip

if [[ $? -ne 0 ]];then
  echo "Invalid IP Address ($ip)"
else
  echo "$ip is a Perfect IP Address"
fi


---

# hasil partisi disk1.img

~~~
dd if=/dev/zero of=disk1.img bs=512M count=8
8+0 records in
8+0 records out
4294967296 bytes (4,3 GB, 4,0 GiB) copied, 54,5068 s, 78,8 MB/s
~~~

fdisk disk1.img
o # dos partition table mbr
g # gpt partition table guid

O # dump disk layout to sfdisk
I # load disk layout from sfdisk
t # change partition type
d # delete partition number

~~~
Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img4      1947648 8388607 6440960  3,1G  5 Extended
disk1.img5      1949696 2359295  409600  200M 83 Linux
disk1.img6      2361344 3282943  921600  450M 83 Linux
disk1.img7      3284992 8388607 5103616  2,4G 83 Linux

Command (m for help): t
Partition number (1,2,4-7, default 7): 7
Partition type (type L to list all types): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto
1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT            
Partition type (type L to list all types): b

Changed type of partition 'Linux' to 'W95 FAT32'.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img4      1947648 8388607 6440960  3,1G  5 Extended
disk1.img5      1949696 2359295  409600  200M 83 Linux
disk1.img6      2361344 3282943  921600  450M 83 Linux
disk1.img7      3284992 8388607 5103616  2,4G  b W95 FAT32
~~~


~~~
fdisk disk1.img

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-8388607, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-8388607, default 8388607): +450M

Created a new partition 1 of type 'Linux' and of size 450 MiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (923648-8388607, default 923648): 
Last sector, +sectors or +size{K,M,G,T,P} (923648-8388607, default 8388607): +500M

Created a new partition 2 of type 'Linux' and of size 500 MiB.

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (3,4, default 3): 
First sector (1947648-8388607, default 1947648): 
Last sector, +sectors or +size{K,M,G,T,P} (1947648-8388607, default 8388607): +200M

Created a new partition 3 of type 'Extended' and of size 200 MiB.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img3      1947648 2357247  409600  200M  5 Extended

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img3      1947648 2357247  409600  200M  5 Extended

Command (m for help): n
Partition type
   p   primary (2 primary, 1 extended, 1 free)
   l   logical (numbered from 5)
Select (default p): l

Adding logical partition 5
First sector (1949696-2357247, default 1949696): 
Last sector, +sectors or +size{K,M,G,T,P} (1949696-2357247, default 2357247): 

Created a new partition 5 of type 'Linux' and of size 199 MiB.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img3      1947648 2357247  409600  200M  5 Extended
disk1.img5      1949696 2357247  407552  199M 83 Linux

Command (m for help): d
Partition number (1-3,5, default 5): 5

Partition 5 has been deleted.

Command (m for help): d
Partition number (1-3, default 3): 3

Partition 3 has been deleted.

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (3,4, default 3): 4
First sector (1947648-8388607, default 1947648): 
Last sector, +sectors or +size{K,M,G,T,P} (1947648-8388607, default 8388607): 

Created a new partition 4 of type 'Extended' and of size 3,1 GiB.

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 5
First sector (1949696-8388607, default 1949696): 
Last sector, +sectors or +size{K,M,G,T,P} (1949696-8388607, default 8388607): +200M

Created a new partition 5 of type 'Linux' and of size 200 MiB.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img4      1947648 8388607 6440960  3,1G  5 Extended
disk1.img5      1949696 2359295  409600  200M 83 Linux

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 6
First sector (2361344-8388607, default 2361344): 
Last sector, +sectors or +size{K,M,G,T,P} (2361344-8388607, default 8388607): +450M

Created a new partition 6 of type 'Linux' and of size 450 MiB.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img4      1947648 8388607 6440960  3,1G  5 Extended
disk1.img5      1949696 2359295  409600  200M 83 Linux
disk1.img6      2361344 3282943  921600  450M 83 Linux

Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 7
First sector (3284992-8388607, default 3284992): 
Last sector, +sectors or +size{K,M,G,T,P} (3284992-8388607, default 8388607): 

Created a new partition 7 of type 'Linux' and of size 2,4 GiB.

Command (m for help): p
Disk disk1.img: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaadfe3c

Device     Boot   Start     End Sectors  Size Id Type
disk1.img1         2048  923647  921600  450M 83 Linux
disk1.img2       923648 1947647 1024000  500M 83 Linux
disk1.img4      1947648 8388607 6440960  3,1G  5 Extended
disk1.img5      1949696 2359295  409600  200M 83 Linux
disk1.img6      2361344 3282943  921600  450M 83 Linux
disk1.img7      3284992 8388607 5103616  2,4G 83 Linux

Command (m for help): w
The partition table has been altered.
Syncing disks.
~~~