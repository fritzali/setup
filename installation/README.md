## Installation

In the following, steps for installing an [Arch Linux](https://archlinux.org/) based system with a simple encrypted partition scheme are listed.

#### Guide

1. Preparation
   1. Acquire the ISO file and PGP signature [here](https://archlinux.org/download/).
   2. Verify the checksum by running <pre>pacman-key -v <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso.sig</pre> from an existing system with the date matching the image version.
   3. Write the ISO to a USB flash drive.

      > **This will delete all data stored on the device, so make sure to secure any important files beforehand.**

      Determine the name of your USB flash drive by running <pre>ls -l /dev/disk/by-id/usb-*</pre> and check `lsblk` to ensure that the device is not mounted. In case any partitions are mounted, run `umount`
      on either the device or the mount point. For the sake of simplicity, there are several equivalent options using `coreutils` commands to write the image:
      1. using the `cat` utility, <pre>cat <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso > /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      2. using the `cp` utility, <pre>cp <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      3. using the `dd` utility, <pre>dd bs=4M if=<i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso of=/dev/disk/by-id/usb-<i>flash_drive</i><br>conv=fsync oflag=direct status=progress</pre>

      > **Executing `sync` with root privileges afterwards ensures buffers are fully written to the device before you remove it.**

      To restore the USB flash drive as an empty storage device, run <pre>wipefs --all /dev/disk/by-id/usb-<i>flash_drive</i></pre> before repartitioning and reformatting.

   4. Boot the live environment.

      > **Secure Boot is not supported and must be diabled during this phase, but can be reenabled after completing the installation.** 

      Point the system to the Arch Linux installation medium. To achieve this, repeatedly press `F2` to enter the BIOS Setup or `F12` to directly set the Boot Device during the post phase.
      Simultaneously, you must either hold down the `Fn` key or have `FnLock` enabled. Key combinations might differ depending on the system. Afterwards, select the Arch Linux installation
      medium when the boot menu appears and press `↵` to enter the live environment.

      > **To enter boot parameters, use `e` with `systemd-boot` for UEFI or `↹` with `syslinux` for BIOS systems.**

      You will be logged in as root user on the first virtual console and presented with a `Zsh` prompt. To switch consoles, use `Alt` with the respective arrow keys. To edit configuration files, both
      `nano` and `vim` are available.

   5. Set the console keyboard layout and font. If necessary, list all available layouts by running <pre>localectl list-keymaps</pre> and load the appropriate option. For example, using
      <pre>loadkeys de-latin1</pre>
      changes from the default `US` English QWERTY to a `DE` German QWERTZ keymap. The directory `/usr/share/kbd/consolefonts/` contains available fonts, which can be set via the `setfont`
      command by omitting the path and file extension.

   6. Make sure the correct boot mode is set by checking the bitness: <pre>cat /sys/firmware/efi/fw_platform_size</pre>
      1. If `64` is returned by the command, your system is booted in UEFI mode with 64bit x64 architecture, which is the ideal scenario.
      2. If it returns `32` instead, the system is booted in UEFI mode with 32bit IA32 architecture. This will limit boot loader choice to those that support mixed mode booting.
      3. In case a message such as `no such file or directory` or similar is returned, the system may be booted in BIOS or CMS mode, which is undesired and should be remedied if possible.

   7. To connect to the internet, first ensure that your network interface is listed and enabled by using <pre>ip link</pre> or a command with similar functionality. For wireless, make sure
      the card is not blocked with `rfkill` and use `iwctl` to authenticate:
      1. Start up an interactive prompt with `iwctl` and run `help` to list all available commands.
      2. Find out your WiFi device name,

         <pre>device list</pre>

      3. If the device or its adapter is turned off, turn it on,

         <pre>device <i>name</i> set-property Powered on</pre>
         <pre>adapter <i>name</i> set-property Powered on</pre>

      4. Initiate a scan for networks, not producing any output,

         <pre>station <i>name</i> scan</pre>

      5. List all available networks,

         <pre>station <i>name</i> get-networks</pre>

      6. Connect to a visible or hidden network and enter the passphrase if prompted,

         <pre>station <i>name</i> connect <i>SSID</i></pre>
         <pre>station <i>name</i> connect hidden <i>SSID</i></pre>

      Alternatively, a single line command is often sufficient as well: <pre>iwctl --passphrase <i>passphrase</i> station <i>name</i> connect <i>SSID</i></pre>
      The connection can be verified by pinging a known address:
      <pre>ping archlinux.org</pre>

   8. Use `timedatectl` to ensure the system clock is synchronized.
   9.
   10.
   11.
   
1. Installation
2. Configuration

*Adapted from the [Arch Wiki](https://wiki.archlinux.org/)*.
