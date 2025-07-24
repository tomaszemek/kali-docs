# Installation

# Bare metal, AMD64

## Installation image and UEFI

Download the installation image and put in on USB stick. 

Recommended to pick a stick smaller than 32 GB, as that's the upper limit for FAT32 partition. Tools like Rufus will default to FAT32 filesystem for the stick but it won't restrict the partition on size, resulting in installer boot failure in GRUB with stick larger than 32 GB.

To put the image onto the USB stick, preferably use a tool that creates read-write filesystem on the stick, so you can perform modifications later if needed. Examples of such tools are Rufus for Windows or SystemRescue USB writer for Linux.

Tools like Etcher for Windows or dd for Linux create read-only filesystem on the USB stick which will work fine unless modifications need to be made in order to get the installation to work.

Later modifications on the stick might be necessary if UEFI can't find respective EFI image to launch as the first step on boot.

UEFI firmware of the machine aims to find a launchable EFI image of the stick. The default fallback file path is /EFI/BOOT/BOOTX64.EFI

In case the stick won't boot due to UEFI can't find EFI bootloader, modify the path on the stick to match the fallback path. It might be case sensitive too.

The image contains two 32 bit and two 64 bit bootloader images, small and large EFI for each. I got the installer to boot successfully with the larger one, 64 bit.

It's important to launch the installer in EFI mode, especially if you want a multi book configuration with other OS's also using EFI boot.

Launching the installer in MBR mode might end up installing GRUB in MBR mode too, as the installer won't be able to detect EFI environment later, making it incompatible with other operating systems.

## Encrypted filesystems

[Encryption](encryption.md)



