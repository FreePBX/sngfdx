<mark>This sngfd12 repo was archived on September 21, 2026.
Please instead see https://github.com/FreePBX/sngfdx 👀</mark>

# abuild-fpbx-installer-iso

"The FreePBX Factory"

### Copyright (C) 2024-2026, Sangoma US Inc.

### GNU General Public License (GPL) v2.0-only

## Authors
- Chris Maj <cmaj+fpbx17iso@sangoma.com>
- Duncan Patterson <dpatterson+fpbx17iso@sangoma.com>
- Mallavolu Purna Sai <spurna+fpbx17iso@sangoma.com>
- Pushkar Singh <psingh+fpbx17iso@sangoma.com>

*Includes many ideas from earlier first-boot.sh by Sirs Purna & Pushkar*

## Purpose
Use this script to generate ISOs that can be used
to rapidly install FreePBX onto a system, using a base
Debian ISO that gets grafted on to bit-by-bit.

## Example
```bash
$ bash abuild-fpbx-installer-iso -i debian-12.8.0-amd64-netinst.iso
```

## Output
1. an ISO ready to write to a USB stick (required: USB stick size less than appx 120G)
2. and (optionally) a PXE boot friendly TAR file for use in cobbler, di-netboot-assistant, etc.

## Prerequisites
1. xorriso ISO image tool + ansible automation platform
```bash
$ apt-get install xorriso ansible
```
2. An upstream Debian GNU/Linux ISO such as netinst:
```bash
$ wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso
```
3. Or instead of netinst in #2, use a DVD image for (potentially) faster installs with more hardware drivers:
```bash
$ apt-get install jigdo-file
$ jigdo-lite https://cdimage.debian.org/debian-cd/current/amd64/jigdo-dvd/debian-12.8.0-amd64-DVD-1.jigdo
```

## Caveats
1. Debian 12 "bookworm" was used to develop, test and run this script; but it might work on others.
2. The "initrd.gz" is only required for non-Debian PXE boot servers eg. cobbler on CentOS 7, Fedora 41, Rocky 9, etc.:
```bash
$ wget -O bookworm-netboot-initrd.gz https://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz
```
3. Proper RAID support on UEFI boots only. (Any time there are two non-NVM drives you'll get a RAID setup.)
4. Minimum disk size when virtual is appx 30G without further adjustments to script defaults eg. see int_lvm_guided_size
5. Minimum disk size when physical via USB install is appx 120G as less than that is treated as a USB stick.

## SNG 7 instructions if you already have Cobbler 2.8 running
Note that you may need to copy over additional RPMs to complete the first `yum install` below.

```bash
yum install grub2 grub2-efi-x64-modules grub2-tools grub2-common grub2-pc grub2-tools-extra grub2-efi-x64 grub2-pc-modules grub2-tools-minimal
wget https://cobbler.github.io/signatures/2.8.x/latest.json
patch -p0 < legacy/bookworm-patch-cobbler-2.8.latest.json.diff
cp latest.json /var/www/html
chmod 755 /var/www/html/latest.json
echo "signature_url: http://localhost/latest.json" >> /etc/cobbler/settings
systemctl restart cobblerd
sleep 10
cobbler signature update
sh legacy/mkloaders.sh
```

...then run each time a new ISO is ready something like the following:

`sh SNGDEB-PBX17-amd64-12-10-0-2503-2-RUN-ON-PXE-COBBLER-SNG7.sh 192.0.2.42`

## WARNINGS
**This ISO strives for Fully Automated Installation (FAI)**
**and you will lose all existing data on any system you boot with it.**

## TODOs
- use "install" instead of "cp" for things
- remove cruft from /var/tmp/ in the sng-eol script (such as leftover systemd temp directories)
- check if the biosgrub is needed anymore in the partitioning
- reduce number of ors to true - errors should be handled better for each command with a notice in the installer log
- crypto at rest version (safer version + LUKS encrypted hard drive with keys on unique USB drive)
- delete packages from ISO that are not used during initial installation (removed gnome/kde but more needed)
- include packages that the shell script installer downloads (some duplicates but archives.tar is pretty good)
- check if archives is actually any faster...
- port archives to the ansible version (still only the legacy shell script)
- simple-cdd (*might* be easier than all the xorriso usage)
- the bool_automatic toggle set to false is not very step-by-step yet
- make int_mb_ssd_wear a percentage so it dynamically changes based on disk size
- partition mounting improvements (several TODOs there)
- genericize the disk types (s, v, xv -> one)
- nvm "disks" like the UC25 do not show up in the preseed chooser but later on
- consider moving FreePBX apt key to external file as an optional include
- perhaps a proper jigdo of the utilized deb packages instead of archive.tar method
- use preseed_fetch instead of tftp (probably needed for cloud if it will be http instead of tftp)
- consider console=hvc0 in the cloud
- 3rd partition that gets blkdiscard'ed may need to be at end of disk to reclaim space
- GRUB serial no go on reboot (needs Debian template modifation in /etc/grub/ ...)
- consider copyright notice at bottom of GRUB wallpaper like SNG7 did
- consider restoring more command-line options from the legacy shell script
- should allow SSH pub keys to be copied in to ISO then installed on target system for sangoma user
