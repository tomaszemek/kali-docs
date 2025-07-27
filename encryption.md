
Last updated: 27-July-2025

# Encryption

## Installing to encrypted volumes

It is possible to encrypt some or all of the volumes at various levels.

Kali installer offers everything in order to create encrypted volumes, however some of the volume encryption parameters cannot be changed from the installer UI. Alternatively, one can create encrypted volumes manually aside of the installer however using such prepared volumes back in the Kali installer doesn't seem to work very straightforward.

Kali installer will also not ask for custom names of the encrypted volumes (as shown later in crypttab) but will use default naming scheme (device_crypt).
That's not a problem, as the names can be changed later on first boot.

## Cryptsetup benchmark

In order to determine the most effective encryption parameter combination for given hardware, the cryptsetup benchmark tool can be run before creating encrypted volumes.

## Encrypted root

If root fs encryption is desired, the encrypted volume (using LUKS2) needs to be created during installation (when at the disk partitioning step of the Kali installer). As GRUB doesn't support booting from LUKS2 encrypted volume, it is recommended to create separate volume for /boot (e.g. 4 GB) and leave it unencrypted during installation and convert it to LUKS1 encrypted volume later, as LUKS1 is bootable by GRUB. Full system encryption is therefore achievable by combination LUKS1 for boot and LUKS2 for root.

### Manual creation of encrypted root volume

This step is optional and serves as an alternative to creating encrypted root filesystem manually, bypassing the Kali installer somehow.
Originally this procedure was used to install Kubuntu, which uses such an installer that allows user to prepare encrypted root volume manually first and then use it by the installer later. However, with Kali it doesn't work that easily.

This example creates encrypted volume named root_kali on device /dev/rootpart (use your own respective device name, like nvme0n1p1) with Ext4 filesystem in 32-bit mode (optimization for size and speed, usable for volumes up to 16 TB size) labelled also root_kali.
It also creates Ext4 fs on boot device /dev/bootpart (use your own respective device name, like sda2).
After installation it updates all the ID's so that the system will mount all fs's on next reboot.

```
(as root)
$ sudo su

(find optimal hash size)
# cryptsetup benchmark

(use the optimal hash size here:)
# cryptsetup luksFormat --hash=sha256 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/rootpart
# cryptsetup luksOpen /dev/rootpart root_kali

(initialize the device with random data)
# dd if=/dev/urandom of=/dev/mapper/root_kali status=progress bs=4M

(create file systems)
# mkfs.ext4 -F -O ^64bit -L root_kali /dev/mapper/root_kali
# mkfs.ext4 -F -O ^64bit -L boot_kali /dev/bootpart

(install Kali to these, install GRUB, don't reboot yet !!!)

(find out UUID of the root_kali volume)
# uuid="$(blkid -o value -s UUID /dev/rootpart)"

(mount target file system tree, assuming /mnt is not mounted yet:)
# mount /dev/mapper/root_kali /mnt
# mount /dev/bootpart /mnt/boot
# mount --bind /dev /mnt/dev
# mount --bind /proc /mnt/proc
# mount --bind /sys /mnt/sys
# mount --bind /dev/pts /mnt/dev/pts

(/dev/efipart - use your actual EFI partition ID)
# mount /dev/efipart /mnt/boot/efi

(enter the new filesystem tree:)
# chroot /mnt

(unlock the encrypted volume on boot:)
(use the discard keyword in the last argument only if your /dev/rootpart is SSD/NVMe drive)
# echo "root_kali UUID=$uuid none luks,discard" | tee -a /etc/crypttab

(update initramfs with the new crypttab)
# update-initramfs -c -k all

(update grub with the new UUIDs)
# update-grub

(all done, safe to reboot)
# reboot

(after reboot, GRUB menu will appear, and system will later ask for passphrase to unlock the root volume)

```

### Customize default encrypted volume name

This step is optional but if performed, it shall be done prior to the following steps.

If encrypted volume was created during installation for the root fs, it will go under the default naming scheme: device_crypt
This can be changed into something more meaningful in /etc/crypttab (set the new volume name) and then in /etc/fstab (use the newly set volume name for the root filesystem). After that, initramfs needs to be updated. There will be complaints during initramfs update, ignore it. On reboot, the root filesystem might fail to mount automatically. Mount it manually, finish boot and run initramfs update once more. This time there will be no more complaints and the next reboot the root fs will be mounted automatically again.

## Encrypted boot

Encrypting boot reduces risk of attack via boot and initramfs files tampering. It is optional step however.

### Automatic mount of encrypted volumes

In order to open the the encrypted boot volume later automatically during boot, the following package is required to be installed, however it is not installed by default: systemd-cryptsetup. It needs to be installed manually at this point. Without this package present, mounting /boot via fstab fails as the boot volume is not opened.

```
$ sudo su
# apt update && apt install systemd-cryptsetup
```

Also, as Kali disables root account if user chose to create own user account during installation, the emergency rescue console that would allow manual opening of the boot volume cannot be launched during boot. One of the ways out if this is to reboot from the hanging boot attempt and modify the kernel cmdline in GRUB, add init=/bin/bash and mount the root filesystem with rw to be able to edit fstab.

### Convert boot to LUKS1

Reboot and make sure everything works automatically as expected. If not fix it now before moving on.

Notes: 

- It might occur that the cryptsetup luksFormat command in the following block refuses to act due to device in use. I tried to find the culprit but haven't. The device was unmounted and there was no open handle to it, yet no chance. I personally suspect VirtualBox to be causing it somehow as without Virtualbox installed there was no issue.

Warning:

- This procedure shall be taken in one go. Do not let the computer sleep or hibernate before completed all steps, as it might not be able to pick where it left off or boot again.

```
(as root)
$ sudo su

(backup /boot contents)
# mount -oremount,ro /boot
# install -m0600 /dev/null /tmp/boot.tar
# umount /boot/efi
# tar -C /boot --acls --xattrs --one-file-system -cf /tmp/boot.tar .
# umount /boot

(create LUKS1 encrypted volume on device /dev/bootpart - use own respective device name)
# cryptsetup luksFormat --type luks1 /dev/bootpart
# cryptsetup luksOpen /dev/bootpart boot_kali

(find out UUID of the boot_kali volume)
# uuid="$(blkid -o value -s UUID /dev/bootpart)"

(unlock the encrypted volume on boot:)
(use the discard keyword in the last argument only if your /dev/bootpart is SSD/NVMe drive)
# echo "boot_kali UUID=$uuid none luks,discard" | tee -a /etc/crypttab

(initialize the device with random data)
# dd if=/dev/urandom of=/dev/mapper/boot_kali status=progress bs=4M

(create file system)
# mkfs.ext4 -F -O ^64bit -L boot_kali /dev/mapper/boot_kali

(update fstab, change old specification of boot partition (likely identified by UUID) to new one (/dev/mapper/boot_kali):)
(example:)
(/dev/mapper/boot_kali  /boot   ext4    defaults    0   2)
# nano /etc/fstab

(apply the change)
# systemctl daemon-reload

(restore /boot from backup:)
# mount -v /boot
# tar -C /boot --acls --xattrs -xf /tmp/boot.tar
# mount -v /boot/efi

(enable encrypted boot:)
# echo "GRUB_ENABLE_CRYPTODISK=y" >> /etc/default/grub
# update-grub

(grub should be reinstalled after configuration change; /dev/bootdevice is typically a full device (like /dev/sda or other), not a partition (like /dev/sda2))
# grub-install /dev/bootdevice

(enable unlocking by keys:)
# mkdir -m0700 /etc/keys

(generate key file for root_kali:)
# ( umask 0077 && dd if=/dev/urandom bs=1 count=64 of=/etc/keys/root_kali.key conv=excl,fsync )

(update root_kali with the new key:)
(show current key slots before:)
# cryptsetup luksDump /dev/rootpart

(add the key:)
(use the same hash size as used during LUKS creation; or check with existing keys in the previous dump)
# cryptsetup luksAddKey --hash=sha256 /dev/rootpart /etc/keys/root_kali.key

(show current key slots after, note the new key slot ID:)
# cryptsetup luksDump /dev/rootpart

(update crypttab:)
(replace none with /etc/keys/root_kali.key)
(add key-slot=n to the list of options
(replace n with the actual ID of the newly added key as shown in the luksDump after adding the key))
(example:)
(root_kali UUID=1ae16424-8587-4575-9f2b-fc62dd3a7a68 /etc/keys/root_kali.key luks,discard,key-slot=1)
# nano /etc/crypttab

(repeat for boot_kali:)
(generate key file for boot_kali:)
# ( umask 0077 && dd if=/dev/urandom bs=1 count=64 of=/etc/keys/boot_kali.key conv=excl,fsync )

(update boot_kali with the new key:)
(show current key slots before:)
# cryptsetup luksDump /dev/bootpart

(add the key:)
(LUKS1 has no hashes)
# cryptsetup luksAddKey /dev/bootpart /etc/keys/boot_kali.key

(show current key slots after, note the new key slot ID:)
# cryptsetup luksDump /dev/bootpart

(update crypttab:)
(replace none with /etc/keys/boot_kali.key)
(add key-slot=n to the list of options
(replace n with the actual ID of the newly added key as shown in the luksDump after adding the key))
(example:)
(boot_kali UUID=6c49db00-eb52-4546-a787-a2091f625c4a /etc/keys/boot_kali.key luks,discard,key-slot=1)
# nano /etc/crypttab

(include root volume key in the initramfs:)
# echo "KEYFILE_PATTERN=\"/etc/keys/*.key\"" >> /etc/cryptsetup-initramfs/conf-hook

(set proper permissions:)
# echo UMASK=0077 >> /etc/initramfs-tools/initramfs.conf

(update initramfs:)
# update-initramfs -u -k all

(locate the initramfs files:)
# stat -L -c "%A  %n" /boot/initrd.img*

(verify that the keys are present in the initramfs files:)
# lsinitramfs /boot/initrd.img* | grep "^cryptroot/keyfiles/"

(all done, safe to reboot)
# reboot

(after reboot, system will prompt password to the boot partition, which will after unlocking show GRUB menu and it will by default boot kernel with root volume being unlocked by key file from the initramfs; boot volume is first unlocked by passphrase during boot, but later again by key by systemd-cryptsetup, prior to mounting the /boot via fstab)
```

## Additional encrypted volumes

Adding more encrypted volumes be like (for more details see above in the 'convert boot to LUKS1'):

1. (optional: cryptsetup benchmark)
2. cryptsetup luksFormat
3. cryptsetup luksOpen
4. initialize with /dev/urandom
5. add to crypttab 
6. mkfs
7. create mount point
8. add fs to fstab
9. generate key
10. cryptsetup luksAddKey
11. update crypttab

Not sure if a device can be encrypted with LUKS on the go without data loss. Probably have to backup and restore.
