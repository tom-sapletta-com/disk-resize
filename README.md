# disk-resize
resize disks sda, nvme0


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



