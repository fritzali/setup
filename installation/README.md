## Installation

In the following, steps for installing an [Arch Linux](https://archlinux.org/) based system with a simple encrypted partition scheme are listed.

#### Guide

1. Preparation
   1. Acquire the ISO file and PGP signature [here](https://archlinux.org/download/).
   2. Verify the checksum by running <code>pacman-key -v <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso.sig</code> from an existing system with the date matching the image version.
   3. Write the ISO to a USB flash drive. **This will delete all data stored on the device.** To determine the name of the USB flash drive, use the command `ls -l /dev/disk/by-id/usb-*` and check
      `lsblk` to ensure that the device is not mounted. In case any partitions are mounted, run `umount` on either the device or the mount point.
      Using basic `coreutils` commands for the sake of simplicity, there are several equivalent options to write the image:
      1. using the `cat` utility: <pre>cat <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso > /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      2. using the `cp` utility: <pre>cp <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      3. using the `dd` utility: 
   5. 
   6. 
      ```
      test
      test
      ```

   7. test `test`
   8. test
2. Installation
3. Configuration

> test

*Adapted from the [Arch Wiki](https://wiki.archlinux.org/)*.
