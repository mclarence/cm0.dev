---
layout: post
title:  "How to Boot PopOS with Secure Boot"
date:   2022-01-06
tags: linux secure-boot popos
comments: true
---

## Intro
---
How to boot PopOS (or any linux distro) with secure boot enabled.

## Prerequisites
---
* This guide assumes that you have a working PopOS (21.10) installation and secure boot is currently disabled in the UEFI BIOS.
* I also assume you have some experience working through the command line in linux.
* These two files, obtainable [here](https://blog.hansenpartnership.com/linux-foundation-secure-boot-system-released/)
  * *Preloader.efi*
  * *Hashtool.efi*

## Steps
---
1. Open a terminal session and gain root privileges.
```
sudo -i
```
2. Change directory to `/boot/efi/EFI/systemd`.
```
cd /boot/efi/EFI/systemd
```
3. Verify that `systemd-bootx64.efi` is in this directory.
```
$> ls
systemd-bootx64.efi
```
4. Rename `systemd-bootx64.efi` to `loader.efi`
```
mv systemd-bootx64.efi loader.efi
```
5. Move both `Preloader.efi` and `HashTool.efi` to `/boot/efi/EFI/systemd`
```
mv Preloader.efi /boot/efi/EFI/systemd
mv HashTool.efi /boot/efi/EFI/systemd
```
6. Rename `Preloader.efi` to `systemd-bootx64.efi`
```
mv Preloader.efi systemd-bootx64.efi
```
7. Reboot into BIOS and enable secure boot.
8. Reboot again and you should now see this screen:
  ![bootimg](/assets/images/secure-boot-popos/step8.jpg)
  Simply select OK to continue.
9. Select *Enroll Hash*
  ![enrollhash](/assets/images/secure-boot-popos/step9.jpg)
10. Select `loader.efi` and press enter.
  ![enrollhash1](/assets/images/secure-boot-popos/step10.jpg)
11. It will ask if you want to *Enroll this hash into the MOK database?*. Simply select YES.
  ![enrollhash2](/assets/images/secure-boot-popos/step11.jpg)
12. It will bring you back to the main menu. Select *Enroll Hash* again.
  ![enrollhash](/assets/images/secure-boot-popos/step9.jpg)
13. Repeat steps 10 and 11 with `vmlinuz.efi` which can be found by selecting `../` > `Pop_OS-....`
14. Once you have completed the above, reboot the system and it should successfully boot into PopOS with secure boot enabled.

## Notes
---
* Sometimes when updating PopOS, it will modify the .efi files. This will prevent PopOS from booting with secure boot again. To fix this simply re-enroll the hashes again (Steps 9-13) to fix the issue.