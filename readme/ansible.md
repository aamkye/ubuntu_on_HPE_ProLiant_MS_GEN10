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

* [ZFS](./zfs.md)
* [LVM](./lvm.md)
* [HPE](./hpe.md)
