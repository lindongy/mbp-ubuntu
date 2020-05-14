# mbp-ubuntu

The ISO in from this repo should allow you to install ubuntu without using an external keyboard or mouse on a MacBook Pro 16,1. It work in my MacBook Pro 16,1.

[![Build Status](https://travis-ci.com/marcosfad/mbp-ubuntu.svg?branch=master)](https://travis-ci.com/marcosfad/mbp-ubuntu)

**If this repo helped you in any way, consider inviting a coffee to the people in the [credits](https://github.com/marcosfad/mbp-ubuntu#credits) or [me](https://paypal.me/marcosfad)**

UBUNTU 20.04 ISO with Apple T2 patches built-in (Macbooks produced >= 2018).

Apple T2 drivers are integrated with this iso. 

This repo is a rework of the great work done by [@mikeeq](https://github.com/mikeeq/mbp-fedora)

I'm using the Kernel from - <https://github.com/marcosfad/mbp-ubuntu-kernel>

Using additional drivers:
- [Apple T2 (apple-bce) (audio, keyboard, touchpad)](https://github.com/MCMrARM/mbp2018-bridge-drv)
- [Touchbar (apple-ibridge, apple-ib-tb, apple-ib-als)](https://github.com/roadrunner2/macbook12-spi-driver/tree/mbp15)

Bootloader is configure correctly out of the box. No workaround needed.

## Installation

1. Reduce the size of the mac partition in MacOS
2. Download ISO file from releases.
3. Copy it to a USB using dd (or gdd if installed over brew): 
```bash
diskutil list # found which number has the USB
diskutil umountDisk /dev/diskX
sudo gdd bs=4M if=ubuntu-20.04-5.6.10-mbp.iso of=/dev/diskX conv=fdatasync status=progress
```
4. Boot in Recovery mode and allow booting unknown OS
5. Restart and immediately press the option key until the Logo come up
6. Select "EFI Boot" (the third option was the one that worked for me)
7. Launch Ubuntu Live
8. Use Ubiquity to install (just click on it)
9. Select the options that work for you and use for the partition the following setup:
    * Leave the efi boot as preselected by the installer. Your Mac will keep on working with out problems.
    * Add a ext4 partition and mounted as /boot (1024MB)
    * Add a ext4 partition and monted as / (rest)
10. Run the installer (In my case it had some problem removing some packages at the end, but this is no real problem)
11. Shutdown and remove the USB Drive
12. Start again using the option key. Select the new efi boot.
13. Enjoy.

## Configuration

- Put wifi firmware files to `/lib/firmware/brcm/`
  - tutorial - <https://github.com/mikeeq/mbp-fedora-kernel/#working-with-mbp-fedora-kernel>
  - this is not working with MBP 16,1
- To install additional languages, install appropriate langpack via apt `sudo apt-get install language-pack-[cod] language-pack-gnome-[cod] language-pack-[cod]-base language-pack-gnome-[cod]-base `
    - see https://askubuntu.com/questions/149876/how-can-i-install-one-language-by-command-line
- You can change mappings of ctrl, fn, option keys (PC keyboard mappings) by creating `/etc/modprobe.d/hid_apple.conf` file and recreating grub config. All available modifications could be found here: <https://github.com/free5lot/hid-apple-patched>
```
# /etc/modprobe.d/hid_apple.conf
options hid_apple swap_fn_leftctrl=1
options hid_apple swap_opt_cmd=1
```
- I switch the touchbar to show f* by default. If you like another configuration, change /etc/modprobe.d/apple-tb.conf or remove it.
- To update grub, run: `grub-mkconfig -o /boot/grub/grub.cfg`

## Update to newer kernels

**IF YOU UPDATE THE KERNEL, REMEMBER TO ADD THE REQUIRED DRIVERS AGAIN.**

### The easy way:
```bash
sudo apt install dkms
sudo apt install linux-headers-<mbp-kernel-release>-mbp linux-image-<mbp-kernel-release>-mbp
sudo git clone --branch mbp15 https://github.com/roadrunner2/macbook12-spi-driver.git /usr/src/apple-ibridge-0.1
sudo dkms install -m apple-ibridge -v 0.1 -k <mbp-kernel-release>-mbp
modprobe apple-ib-tb
modprobe apple-ib-als
sudo git clone --branch aur https://github.com/marcosfad/mbp2018-bridge-drv.git /usr/src/apple-bce-0.1
sudo dkms install -m apple-bce -v 0.1 -k <mbp-kernel-release>-mbp
modprobe apple-bce
```

### Another way:
Check <https://github.com/marcosfad/mbp-ubuntu/blob/master/files/chroot_build.sh> to see how it is done.


## Know issues

- Sound is not working after the install. Follow the instructions detailed by @kevineinarsson: <https://gist.github.com/kevineinarsson/8e5e92664f97508277fefef1b8015fba> (it didn't worked for me on MBP 16,1, but I don't use sound)
- Wifi is not working for bali (MBP 16,1). I'm using a 10$ usb wifi.
- Checksum is failing for 2 files: md5sum.txt and /boot/grub/bios.img
- I'm having troubles shutting down ubuntu. Screen is black but fan keeps on working. I have to force shutdown. 

## Not working (Following the mikeeq/mbp-fedora)

- Dynamic audio input/output change (on connecting/disconnecting headphones jack)
- TouchID - (@MCMrARM is working on it - https://github.com/Dunedan/mbp-2016-linux/issues/71#issuecomment-528545490)
- Thunderbolt (is disabled, because driver was causing kernel panics (not tested with 5.5 kernel))
- Microphone (it's recognised with new apple t2 sound driver, but there is a low mic volume amp)

## TODO

- ISO is using gzip initramfs. It would be great to change it lz4
- Optimize the software installed.

## Known issues (Following the mikeeq/mbp-fedora)

- Kernel/Mac related issues are mentioned in kernel repo
- `ctrl+x` is not working in GRUB, so if you are trying to change kernel parameters - start your OS by clicking `ctrl+shift+f10` on external keyboard

## Docs

- Discord: <https://discord.gg/Uw56rqW>
- WiFi firmware: <https://packages.aunali1.com/apple/wifi-fw/18G2022>
- Linux on a MBP Late 2016: <https://gist.github.com/gbrow004/096f845c8fe8d03ef9009fbb87b781a4>
- Repack Bootable ISO: <https://wiki.debian.org/RepackBootableISO>
- <https://github.com/syzdek/efibootiso>

### Ubuntu

- <https://help.ubuntu.com/community/LiveCDCustomization>
- <https://itnext.io/how-to-create-a-custom-ubuntu-live-from-scratch-dd3b3f213f81>
- <https://help.ubuntu.com/community/LiveCDCustomizationFromScratch>
- <https://help.ubuntu.com/community/InstallCDCustomization>
- <https://linuxconfig.org/legacy-bios-uefi-and-secureboot-ready-ubuntu-live-image-customization>

### Github

- GitHub issue (RE history): <https://github.com/Dunedan/mbp-2016-linux/issues/71>
- VHCI+Sound driver (Apple T2): <https://github.com/MCMrARM/mbp2018-bridge-drv/>
- hid-apple keyboard backlight patch: <https://github.com/MCMrARM/mbp2018-etc>
- alsa/pulseaudio config files: <https://gist.github.com/MCMrARM/c357291e4e5c18894bea10665dcebffb>
- TouchBar driver: <https://github.com/roadrunner2/macbook12-spi-driver/tree/mbp15>
- Kernel patches (all are mentioned in github issue above): <https://github.com/aunali1/linux-mbp-arch>
- ArchLinux kernel patches: <https://github.com/ppaulweber/linux-mba>
- ArchLinux installation guide: <https://gist.github.com/TRPB/437f663b545d23cc8a2073253c774be3>
- hid-apple-patched module for changing mappings of ctrl, fn, option keys: <https://github.com/free5lot/hid-apple-patched>
- Audio configuration: <https://gist.github.com/kevineinarsson/8e5e92664f97508277fefef1b8015fba>
- Ubuntu in MBP16: <https://gist.github.com/gbrow004/096f845c8fe8d03ef9009fbb87b781a4>

## Credits

- @mikeeq - thanks for the amazing work in mbp-fedora
- @MCMrARM - thanks for all RE work
- @ozbenh - thanks for submitting NVME patch
- @roadrunner2 - thanks for SPI (touchbar) driver
- @aunali1 - thanks for ArchLinux Kernel CI
- @ppaulweber - thanks for keyboard and Macbook Air patches
- @kevineinarsson - thanks for the audio settings