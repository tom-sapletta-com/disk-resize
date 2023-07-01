# [disk-resize](https://tom-sapletta-com.github.io/disk-resize/)

resize disks sda, nvme0

+ [8 Linux 'Parted' Commands to Create, Resize and Rescue Disk Partitions](https://www.tecmint.com/parted-command-to-create-resize-rescue-linux-disk-partitions/)
+ [How to Resize LVM Partition in Linux](https://linuxopsys.com/topics/resize-lvm-partition-in-linux)
+ [Recover From Grub Failure - Proxmox VE](https://pve.proxmox.com/wiki/Recover_From_Grub_Failure)


## error: disk `lvmid/8svGLw-BMJO-oHsj-gpkr-iax1-Yy7c-tPb0av/X6aAE9-IkU4-Rdzo-0fJ0-Xy99-Btwq-xSmzDe' not found.

+ [Super Grub2 Disk download | SourceForge.net](https://sourceforge.net/projects/supergrub2/)
+ [Super Grub2 Disk](https://www.supergrubdisk.org/super-grub2-disk/)
Use Super-Grub2 to boot:
 
- boot from a Super_grub_disk2 rescue iso -> https://www.supergrubdisk.org/
- An orange colored menu appears
- Select: "Enable GRUB2's RAID and LVM support" [ENTER]
- Press ESC when finished to go back to the main menu
- Select: "Boot manually" [ENTER]
- Select: "Operating Systems" [ENTER]
- Scroll down to the second last option: "Linux /boot/vmlinuz-5.xx.xx-x-pve (lvm/pve-root)" [ENTER]
- Your system will reboot and boots into Proxmox VE as before.
- Inside Proxmox VE shell: update-grub to repair GRUB2
 
the last step (update-grub) threw errors after booting about disks not found
```
lvextend -L +1G /dev/pve/root
lvreduce -L -1G /dev/pve/root
```

Proxmox VE shell: 
```
update-grub
```


## check partitions

```
fdisk -l
```

![image](https://github.com/tom-sapletta-com/disk-resize/assets/5669657/9d6adbb9-a14f-4e6c-8740-1cb3a615bf50)


another:
```
lsblk
df -h
```

## resize

+ [lvresize(8): resize logical volume - Linux man page](https://linux.die.net/man/8/lvresize)

Change or set the logical volume size in units of logical extents.  With the + or - sign the  value  is added to or subtracted from the actual size of the logical volume and without it, the value is taken as an absolute one. 
```
-l, --extents [+|-]LogicalExtentsNumber[%{VG|LV|PVS|FREE|ORIGIN}]    
```

Change or set the logical volume size in units of megabytes.  A size suffix of M for megabytes, G  for gigabytes, With the + or - sign the value is added or subtracted from the actual size of the logical volume and rounded to the full extent size and without it, the value is taken as an absolute one.
```    
-L, --size [+|-]LogicalVolumeSize[bBsSkKmMgGtTpPeE]
```

### ROOT
```
lvresize -L -5G /dev/pve/root
resize2fs /dev/mapper/pve-root
```


### LVM DATA


+ [vgdisplay: attribs of volume groups - Linux man page](https://linux.die.net/man/8/vgdisplay)
display attributes of volume groups 
```
vgdisplay
```

![image](https://github.com/tom-sapletta-com/disk-resize/assets/5669657/b0a0e025-e699-4543-8264-20a45045b3d0)



+ [resize2fs - Linux man page](https://linux.die.net/man/8/resize2fs)
-f     Forces resize2fs to proceed with the filesystem resize operation, overriding some safety checks which resize2fs normally enforces. 

```
lvresize -l +100%FREE /dev/pve/data
resize2fs /dev/mapper/
resize2fs /dev/mapper/pve-root
```

```
e2fsck -fy /dev/nvme0n1p3 
resize2fs -f /dev/nvme0n1p3
```

```
lvresize -L -5G /dev/pve/root
resize2fs /dev/mapper/pve-root
```


## LVM DATA INFO


To see the more detail of our VG
```
vgdisplay -v
```

```
lvdisplay
```

+ [lvdisplay: attribs of logical volume - Linux man page](https://linux.die.net/man/8/lvdisplay)
Details of Volume Groups  
```
lvdisplay
```



## resizing data to 650G and give the rest to pve/root

# Remove pve-data logical volume.
```
lvremove /dev/pve/data -y
```

# Create it again with a new size.
```
lvcreate -L 650G -n data pve -T
```

Give pve-root all the other size.
```
lvresize -l +100%FREE /dev/pve/root
```

Resize pve-root file system
```
resize2fs /dev/mapper/pve-root
```



Check disk space before
```
df -h
```      


Delete local-lvm storage in gui
```
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

Check disk space after
```
df -h
```




## reduce /dev/pve/data to 10GB, the following will work without any problem:

```
umount /var/lib/vz
e2fsck -f /dev/pve/data
resize2fs /dev/pve/data 10G
lvreduce -L 10G /dev/pve/data
resize2fs /dev/pve/data
mount /dev/pve/data
```



## resize max /dev/mapper/pve-root

```
du -sh /*
lvextend --extents +100%FREE /dev/mapper/pve-root
resize2fs -p /dev/mapper/pve-root
```


## another

```
lvresize -L +195G /dev/pve/root
# Resize pve-root file system
resize2fs /dev/mapper/pve-root
```




```
lvm vgchange -a y
e2fsck -f /dev/mapper/pve-root
resize2fs -f /dev/mapper/pve-root 195G
```




# Openwrt


Parted
+ [8 Linux 'Parted' Commands to Create, Resize and Rescue Disk Partitions](https://www.tecmint.com/parted-command-to-create-resize-rescue-linux-disk-partitions/)

[[HOWTO] Resizing root partition on x86 - Installing and Using OpenWrt - OpenWrt Forum](https://forum.openwrt.org/t/howto-resizing-root-partition-on-x86/140631)

```sh
BOOT="$(sed -n -e "/\s\/boot\s.*$/{s///p;q}" /etc/mtab)"
DISK="${BOOT%%[0-9]*}"
PART="$((${BOOT##*[^0-9]}+1))"
ROOT="${DISK}${PART}"
LOOP="$(losetup -f)"
losetup ${LOOP} ${ROOT}
fsck.ext4 -y -f ${LOOP}
resize2fs ${LOOP}
reboot
```

script: 
resize.sh

```sh
#!/bin/sh
echo -n "$(sed -n -e "/\s\/boot\s.*$/{s///p;q}" /etc/mtab)" > bootdisk.txt
BOOT=$(cat bootdisk.txt)
echo $BOOT

DISK=${BOOT%%[0-9]*}
PART=$((${BOOT##*[^0-9]}+1))
echo -n "${DISK}" > rootdisk.txt
echo -n "${PART}" >> rootdisk.txt
ROOT=$(cat rootdisk.txt)
echo $ROOT

LOOP=$(losetup -f)
echo ${LOOP} ${ROOT}

losetup ${LOOP} ${ROOT}
fsck.ext4 -y -f ${LOOP}
resize2fs ${LOOP}
reboot
```




# GRUB problem

[Stuck at grub rescue after an update and reboot. Any tips before I reinstall? : r/Proxmox](https://www.reddit.com/r/Proxmox/comments/vy33ho/stuck_at_grub_rescue_after_an_update_and_reboot/)


After several tests I no longer get the error.
Now I need to start the normal grub menu.

open systemrescuecd usb :

e2fsck -ff /dev/pve/root
resize2fs /dev/pve/root 95G # size 96G - 1G
lvreduce -L -1G /dev/pve/root

modprobe efivarfs
mount /dev/mapper/pve-root /mnt
mount -t proc proc /mnt/proc
mount -t sysfs sys /mnt/sys
mount -o bind /dev /mnt/dev
mount -t devpts pts /mnt/dev/pts/
chroot /mnt
# run inside the chroot:
mount /dev/sda2 /boot/efi
mount -t efivarfs efivarfs /sys/firmware/efi/efivars

update-grub
grub-install /dev/sda



# [Proxmox GRUB lvmid not found. – HITOHA.もえ](https://www.hitoha.moe/proxmox-grub-lvmid-not-found/)

> So, my **EFI Partition** for NVME Disk plugged on **M.2_1** on the motherboard would be:
> 
>     /dev/nvme0n1p2
> 
> Then I do this:
> 
>     mount /dev/nvme0n1p2 /boot/efi
> 
> For UEFI user, do this, it will update your motherboard boot entry:
> 
>     mount -t efivarfs /efivarfs /sys/firmware/efi/efivars
> 
> Then, run update GRUB:
> 
>     update-grub
> 
> Install GRUB to Boot/EFI Partition:
> 
>     grub-install /dev/nvme0n1p2
> 
> Exit `chroot` then reboot!
