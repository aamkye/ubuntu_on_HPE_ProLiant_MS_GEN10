# Helpful ZFS stuff

## Aliases

```bash
# Alias for loading password and mounting
function zload() {
  sudo zfs load-key -L prompt $1 && sudo zfs mount $1
}

# Alias for unloading password and unmounting
function zunload() {
  sudo zfs unmount -f $1 && sudo zfs unload-key $1
}
```

## Create a new ZFS array

[Here's](https://wiki.archlinux.org/title/ZFS/Virtual_disks) a great overview on the core features of ZFS, also [this](https://linuxhint.com/zfs-concepts-and-tutorial/) might help.

Now let's create a `ZFS` array on the `HPE ProLiant MicroServer Gen10`.

List them:

```bash
$ lsblk -d
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
...
sda            8:0    0  3.8T  0 disk
sdb            8:16   0  3.8T  0 disk
sdc            8:32   0  3.8T  0 disk
sdd            8:48   0  3.8T  0 disk
...
```

If you are migrating from old CloudOS and RAID was setup, you need to do following (wipeout raid info and format disks):

**WARNING THIS STEP IS IRREVERSIBLE, PROCEED AT YOUR OWN RISK**

```bash
# For each disk
sudo wipefs --all --force /dev/sd[a-d]

# For each disk (plus following commands)
sudo fdisc /dev/sd[a-d]

# This step is optional
sudo reboot
```

Create a mirror pool over `/dev/sda` and `/dev/sdb` based on the [Ubuntu Tutorial](https://ubuntu.com/tutorials/setup-zfs-storage-pool#1-overview).

```bash
sudo zpool create media mirror /dev/sda /dev/sdb
```

Alternatively, create a `raidz` pool over 4 disks.

This is similar to a `RAID5` pool, using 1 disk for parity.

```bash
sudo zpool create media raidz /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

Or alternatively, create a `raidz2` pool over 4 disks.

This is similar to a `RAID6` pool, using 2 disk for parity.

```bash
sudo zpool create media raidz2 /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

In order to use it, you need to create a file system (also called dataset) on the `zpool`.

This is similar to a 'share' in the CloudOS.

Here's an example

```bash
# The file system gets mounted automatically at /media/pictures.
sudo zfs create media/pictures
```

or if you used my FreeNAS image to create a `ZFS` array

```bash
sudo zpool import
```

Follow the instructions.

## Wipes whole zfs datasets and pools

zfs destroy -r nas


## Existing ZFS pool

It may happen that You already have old ZFS pool:

```bash
TASK [zfs : Create ZFS pool] ***************************************************
fatal: [0.0.0.0]: FAILED! => changed=true
  cmd: zpool create -o autoexpand=on -o autoreplace=on -o delegation=on -o dedupditto=1.5 -o failmode=continue -o listsnaps=on nas raidz2 /dev/sda /dev/sdb /dev/sdc /dev/sdd
  msg: non-zero return code
  rc: 1
  stderr: |-
    invalid vdev specification
    use '-f' to override the following errors:
    /dev/sda1 is part of potentially active pool 'nas'
    /dev/sdb1 is part of potentially active pool 'nas'
    /dev/sdc1 is part of potentially active pool 'nas'
    /dev/sdd1 is part of potentially active pool 'nas'
  stderr_lines: <omitted>
  stdout: ''
  stdout_lines: <omitted>
```

In that situation just type:

```bash
$ sudo zpool import
   pool: nas
     id: 0000000000000000000
  state: ONLINE
status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-EY
 config:

        nas         ONLINE
          raidz2-0  ONLINE
            sda     ONLINE
            sdb     ONLINE
            sdc     ONLINE
            sdd     ONLINE
$ sudo zpool import -f 0000000000000000000
$ zload nas/data
```

## Existing ZFS pool, but the hard-way

Sometimes list and import might not recognize existing pool, so you need to force it to find one.

```bash
> lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
(...)
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

## Changing password for zfs dataset

_Existing keys has to be loaded._

```bash
zfs change-key \
  -o keylocation=prompt \
  -o keyformat=passphrase \
  nas/data
```

## ZFS native encryption

Additionally you can setup encryption on dataset.

```bash
sudo zfs create -o encryption=aes-256-gcm -o keylocation=prompt -o keyformat=passphrase media/pictures
```

After reboot:

```bash
sudo zfs mount -a
```

More info [here](https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/>).
