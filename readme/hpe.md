
TURNOFF BIOS FLAG

```bash
> lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  61.9M  1 loop /snap/core20/1405
loop1                       7:1    0  79.9M  1 loop /snap/lxd/22923
loop2                       7:2    0  44.7M  1 loop /snap/snapd/15534
loop3                       7:3    0  44.7M  1 loop /snap/snapd/15904
loop4                       7:4    0  61.9M  1 loop /snap/core20/1494
sda                         8:0    0 223.6G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0 221.6G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   100G  0 lvm  /
sdb                         8:16   0   3.6T  0 disk
├─sdb1                      8:17   0   3.6T  0 part
└─sdb9                      8:25   0     8M  0 part
sdc                         8:32   0   3.6T  0 disk
├─sdc1                      8:33   0   3.6T  0 part
└─sdc9                      8:41   0     8M  0 part
sdd                         8:48   0   3.6T  0 disk
├─sdd1                      8:49   0   3.6T  0 part
└─sdd9                      8:57   0     8M  0 part
sde                         8:64   0   3.6T  0 disk
├─sde1                      8:65   0   3.6T  0 part
└─sde9                      8:73   0     8M  0 part
```

```bash
> sudo zpool import -f -d /dev/sdb1 abc
cannot import 'abc': one or more devices is currently unavailable
> sudo zpool import -f -d /dev/sdb1 -d /dev/sdc1 -d /dev/sdd1 -d /dev/sde1 abc
> zpool list
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
abc   14.5T  5.05T  9.49T        -         -     1%    34%  1.00x    ONLINE  -
```
