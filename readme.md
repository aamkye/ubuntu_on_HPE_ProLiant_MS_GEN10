# Setting up Ubuntu Server on HPE ProLiant MicroServer Gen10

_Disclaimer: do this at your own risk. No fancy web gui here (except monitoring), just raw unix power._

[![HPE ProLiant MicroServer Gen10](./img/microservergen10.jpeg)](./img/microservergen10.jpeg)

[Original project](https://github.com/aamkye/ubuntu_on_WD_PRx100) was about setting up Ubuntu Server on WD PR4100. Some time passed, so LET'S upgrade!

## Overview

This tutorial covers how to install Ubuntu Server on HPE ProLiant MicroServer Gen10.

It goes from preparation, downloading required packages, running installation, initial configuration and extras that most likely are intended to be used.

---

### But this time, lets monitor it like a pro!

[![node](./img/node.png)](./img/node.png)
[![smart](./img/smart.png)](./img/smart.png)
[![zfs](./img/zfs.png)](./img/zfs.png)


---

## Ansible (automated way)

There is ansible automatization of most steps and more.

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

## Suggested addon:

HPE 870212-B21 - SSD converter kit:

[![HPE 870212-B21](./img/hpe-870212-B21.jpg)](./img/hpe-870212-B21.jpg)

* https://buy.hpe.com/us/en/options/cables-cable-kits/server-enablement-kits/server-enablement-kits/hpe-enablement-kits/hpe-microserver-gen10-sff-nhp-sata-converter-kit/p/870213-B21

---

## Links:

* https://www.hpe.com/psnow/doc/a00008701enw
* TBD

---

## Requirements

* USB flash drive (8GB+)
* Ansible

---

# Ansible
## Pre-requirements

Browse `vars/` folder and do necessary changes like:
  * `main`:
    * `macaddress`
    * `authorized_keys`
    * `hostname`
    * `username`
    * `password`
  * `zfs`:
    * `zfs_pools`
    * `zfs_datasets`
    * `zfs_dataset_pass`

as well as:
  * `ansible.cfg`:
    * `remote_user`

## Config generation

```bash
ansible-playbook autoinstall-generator.yml
```

Then take `./tmp/user-data` file and place it on http(s) server.

## Get ISO

Download chosen iso from [here](https://ubuntu.com/download/server).

## BIOS part

Boot into BIOS and turn off `BIOS_CHECK_NAME` (raid thing).

## Pendrive part

Make bootable usb and boot.

## Boot part (with pendrive)

While booting up, press `e` key to edit grub settings from:

```bash
setparams ' Try or Install Ubuntu Server'

set gfxpayload=keep
linux /casper/vmlinuz ---
initrd /casper/initrd
```

and append some changes:

```bash
setparams ' Try or Install Ubuntu Server'

set gfxpayload=keep
linux /casper/vmlinuz --- autoinstall ds=nocloud-net\;s=http://<ip>:<port>/<location-of-user-data-folder>
initrd /casper/initrd
```

Hit `F10` and then auto install should start.

## Ansible part (optional)

Just run:

```bash
ansible-playbook all_in_one.yml --ask-become-pass -e "target=10.0.0.101 custom_dns=true" -i 10.0.0.101,
```

And watch the magic.

After it's done, it's done :)

## Extras

* [ZFS](./readme/zfs.md)
* [LVM](./readme/lvm.md)
* [HPE](./readme/hpe.md)
