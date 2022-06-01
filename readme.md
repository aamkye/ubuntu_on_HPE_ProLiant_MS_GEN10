# Setting up Ubuntu Server on HPE ProLiant MicroServer Gen10

_Disclaimer: do this at your own risk. No fancy web gui here, just raw unix power._

[![WD PR4100](./img/microservergen10.jpeg)](./img/microservergen10.jpeg)

## TOC

* [Setting up Ubuntu Server on HPE ProLiant MicroServer Gen10](#setting-up-ubuntu-server-on-hpe-pro-liant-micro-server-gen10)
  * [TOC](#toc)
  * TBD

---

## Original [project](https://github.com/aamkye/ubuntu_on_WD_PRx100)..

.. was about setting up Ubuntu Server on WD PR4100. Some time passed, so LET'S upgrade!

## Overview

This tutorial covers how to install Ubuntu Server on HPE ProLiant MicroServer Gen10.

It goes from preparation, downloading required packages, running installation, initial configuration and extras that most likely are intended to be used.

---

## Ansible (automated way)

There is [ansible](./ansible) folder with automatization of all steps from [extras](#extras-meant-to-be-run-on-nas-directly) and more.

---

## HPE ProLiant MicroServer Gen10 Spec

```
* Release date: 2016
* CPU: AMD Opteron™ X3216 Processor (1.6-3.0GHz/2 compute cores/4 graphic cores/1MB/12-15W)
* RAM: 8GB (1 x 8GB) PC4-2400T DDR4 UDIMM / 32GB (2 x 16GB) PC4-2400T DDR4 UDIMM
* USB: 4 x 3.0 ports + 2 x 2.0 ports
* Bays: 4 x 3.5" SATA III (over RAID controller) + Internal SATA III port
  Maximum Internal Storage Non-hot plug SATA 16TB (4 x 4TB)
* LAN: 2 x 1 Gbit/s Ethernet
```

---

## Links:

* https://www.hpe.com/psnow/doc/a00008701enw
* TBD

---

## Requirements

* USB flash drive (8GB+)
* TBD

---

## Download the Ubuntu Server

Download chosen iso from [here](https://ubuntu.com/download/server).

---

## Main process

**Boot**

**Turn off `BIOS_CHECK_NAME`**

**Install**

**Boot up and enjoy!**

---

## Extras (meant to be run on NAS directly)

Now you can SSH to your NAS and start installing extras.

---

### Create a new ZFS array

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

### ZFS native encryption

Additionally you can setup encryption on dataset.

```bash
sudo zfs create -o encryption=aes-256-gcm -o keylocation=prompt -o keyformat=passphrase media/pictures
```

After reboot:

```bash
sudo zfs mount -a
```

More info [here](https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/>).

### ZFS import existing zpool

```bash
> lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
(...)
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

---

## Remarks

* TOC created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc).
* Linted by [markdownlint-cli](https://github.com/igorshubovych/markdownlint-cli)
