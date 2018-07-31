# wecouldntcompletetheupdates
Prevent the failing of Windows 10 upgrades on dual/multi-boot BIOS systems

```
"We couldn't complete the updates
Undoing changes
Don't turn off your computer"
```

## symptom
When doing larger Windows 10 inter release/version upgrades for example 1607 to 1703 the following message loops when (re)trying to install Windows upgrades on dual/multi-boot BIOS systems.

## cause
Missing active boot flag on the Windows partition itself, since the grub2bootloader linux partition uses this flag by default after a custom dualboot install.

## quickfix
Active the partition on which Windows is installed before starting the Windows upgrade with the following command prompt commands:

```
diskpart
sel disk X
sel part 1
active
exit
```

After Windows 10 upgrade was succesfull, reverse the previous commands by making the linux partition (sel part 2) active again!

## comprehensivefix
This assumes the following parition layout giving Windows the first ntfs partition and having it installed after all other OS/bootloaders's have been installed:

```
msdos
p1 ntfs
p2 ext4/btrfs/linux /
p3 doasyoulike
p4 doasyoulike(p5-16) 
```

### Linux Install disk layout
Prepare disk with gparted, device create partition table msdos (1st ntfs /2nd linux) and during linux install select bootloader to install into mbr of that disk (default behaviour for ubuntu 16.04).
After first boot into linux re-install grub2bootloader into the partition instead of mbr and install seperate new mbr code which jumps to the grubbootloader on the 2nd partition without having it active so the ntfs partition could stay active for Windows.

```
sudo dpkg-reconfigure grub-pc #uncheck mbr, check partition of linux install
sudo update-grub2
sudo apt-get install mbr
sudo fdisk /dev/sdX #press p and toggle/disable current 2nd active partition and activate 1st partition
sudo install-mbr /dev/sdX -p 2 #install mbr code which jumps to partition 2
#backup correct prewinmbr
sudo dd if=/dev/sdX of=~/Desktop/sdaprewinactwin.mbr bs=512 count=1
sudo reboot
#If reboot worked fine go ahead and install Windows10
 
```

### Windows Disk Signature
After Windows 10 install and first boot, you'll notice it won't show the grub2bootloader anymore since windows has overwritten most of the mbr with its own bootcode. Don't worry grub2bootloader is still there in the 2nd partition. In Windows 10 go to command prompt and make the linux partition active this will temporary break windows upgrades but fixes booting into linux where we can make a permantent patched/fixed mbr.

### PostWindows boot into linux
Backup the windows altered MBR first which includes a unique Windows Disk Signature. Patch the former pre-windows mbr code with the unique hash from the current mbr code. Write the patched mbr code to the disk and you should have a Windows10/Linux unbreakable upgradable disk layout.
 
```
sudo dd if=/dev/sdX of=~/Desktop/sdapostwinactlin.mbr bs=512 count=1
#dd seek/skip ? sdapatched.mbr
cp sdaprewinactwin.mbr sdapatched.mbr
sudo dd if=sdapostwinactlin.mbr of=sdapatched.mbr skip=440 seek=440 bs=1 count=4 conv=notrunc
sudo dd if=sdapatched.mbr of=/dev/sdX bs=512 count=1 conv=sync
#reboot and check linux/windows booting and windows should be able to upgrade.
```

## LINKS
[MBR info](https://thestarman.pcministry.com/asm/mbr/W7MBR.htm)
