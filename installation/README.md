## Installation

In the following, steps for installing an [Arch Linux](https://archlinux.org/) based system with a simple encrypted partition scheme are listed. To get the most recent firmware versions, it might be useful to
first install [Microsoft Windows](https://www.microsoft.com/en-us/windows) to download any available updates and activate your hardware.

### Main Guide

1. Preparation
   1. Acquire the ISO file and PGP signature [here](https://archlinux.org/download/).
   2. Verify the checksum by running <pre>pacman-key -v <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso.sig</pre> from an existing system with the date matching the image version.
   3. Write the ISO to a USB flash drive.

      > **This will delete all data stored on the device, so make sure to secure any important files beforehand.**

      Determine the name of your USB flash drive by running <pre>ls -l /dev/disk/by-id/usb-*</pre> and check `lsblk` to ensure that the device is not mounted. In case any partitions are mounted, run `umount`
      on either the device or the mount point. For the sake of simplicity, there are several equivalent options using `coreutils` commands to write the image:
      1. Using the `cat` utility, <pre>cat <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso > /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      2. Using the `cp` utility, <pre>cp <i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso /dev/disk/by-id/usb-<i>flash_drive</i></pre>
      3. Using the `dd` utility, <pre>dd bs=4M if=<i>path</i>/<i>to</i>/archlinux-<i>yyyy.mm.dd</i>-x86_64.iso of=/dev/disk/by-id/usb-<i>flash_drive</i><br>conv=fsync oflag=direct status=progress</pre>

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

   8. Run `timedatectl` to ensure the system clock is synchronized.
   9. After your disks are recognized, the live system assigns them to a block device like `/dev/sda` for SATA storage or `/dev/nvme0n1` for NVMe PCIe memory, as well as `/dev/mmcblk0` for eMMC SD
      cards. Identify these devices by running <pre>fdisk -l</pre> before partitioning the drives, which will erase all stored data.

      > **Different physical and logical sector sizes may lead to suboptimal performance, especially with LUKS encryption. As the overhead is likely not noticable under normal usage on modern hardware
      and because modifications from the defaults could introduce instability, no changes are made.**

      An existing partition layout can be checked for by specifying a storage device: <pre>fdisk -l /dev/nvme0n1</pre>

      > **If present, it can be advisable to reuse and adapt a partitioning structure in order to prevent corrupting other bootloaders or similar. To avoid impacting the durability of the SSD and because
      TRIM will be used on the system to reduce performance degradation, no memory cell clearing is performed.**

      Now, open an `fdisk` console and create a partition table on the chosen drive: <pre>fdisk /dev/nvme0n1</pre>

      1. Show the help menu that lists all available commands, `m`
      2. Initialize an empty GPT on the drive, `g`
      3. Create a new partition, `n`
      4. Select `1` as the default number, `↵`
      5. Select the default starting sector, `↵`
      6. Choose a partition size no less than 1MiB to ensure proper alignment. Here, a shift of 512MiB is used for the last sector, `+512M`
      7. Change the partition type, `t`
      8. Enter the alias for an EFI system, `uefi`
      9. Create another partition, `n`
      10. Select `2` as the default number, `↵`
      11. Select the default first sector, `↵`
      12. To use the remaining space, choose the default final sector, `↵`
      13. If required, change the partition type, `t`
      14. Enter the alias for a Linux filesystem, `linux`
      15. Review the GPT by printing, `p`
      16. Decide to abort, `q`
      17. Decide to write, `w`

      <br>

      > **In this partitioning scheme, only the EFI partition and a root directory are accounted for. This practically eliminates the need for later repartitioning and does not hinder recoverability
      thanks to the respective Arch Linux helper tools. A swap file will facilitate hibernate functionality while maintaining flexibility and security.**

   10. After this, the partitions should be prepared for LUKS encryption, formatting and mounting.

       > **Due to TRIM being used, a secure erasure of the drive is omitted. While TRIM measurably enhances SSD performance thanks to sectors being marked as empty by the system on delete instead of
       waiting for a new write request, the data stored on the disk no longer looks entirely random to outside observers. Cleared blocks can be distinguished from regions that store ordered information,
       although the latter is still encrypted. Block patterns can be exploited as metadata to recognize the employed filesystem and possibly even file types. The TRIM command is directly forwared to the
       lower level ciphertext device, which fills the space with either zeroes or ones. On the higher level plaintext device, discarded regions will look like random noise. Upper layer software must
       therefore not rely on the reported ones or zeroes. On the other hand, running TRIM can protect header modifications.**

       Prepare the root partition to be encrypted:
  
       <pre>cryptsetup -v luksFormat /dev/nvme0n1p2</pre>
       <pre>cryptsetup open /dev/nvme0n1p2 root</pre>

       Specify the LUKS version by adding the `--type` flag after `luksFormat` with either the `luks1` or `luks2` option.

       > **Never use filesystem repair software such as `fsck` directly on encrypted volumes. If such tools are not used on decrypted devices, any chance at recovering the key will be lost. Unlocking
       with GRUB takes special attention. Different setups with TPM and Secure Boot are not attempted to keep things simple.**

       Create a filesystem on the unlocked LUKS device and mount the root volume:
  
       <pre>mkfs.ext4 /dev/mapper/root</pre>
       <pre>mount /dev/mapper/root /mnt</pre>

       > **Aiming for simplicity, a journaling `ext4` filesystem is chosen over copy on write alternatives. Such implementations, with `ZFS` and `btrfs` being examples, tend to encompass features
       similar to RAID and LVM that are not necessary in the present case and can even lead to complications under encrypted volumes.**

       Be careful to check that the mapping works as intended:

       <pre>umount /mnt</pre>
       <pre>cryptsetup close root</pre>
       <pre>cryptsetup open /dev/nvme0n1p2 root</pre>
       <pre>mount /dev/mapper/root /mnt</pre>

       Following this, format the unencrypted boot partition only if it has been newly created, and mount it:

       <pre>mkfs.fat -F32 /dev/nvme0n1p1</pre>
       <pre>mount --mkdir /dev/nvme0n1p1 /mnt/boot</pre>

   11. Enabling TRIM support on LUKS2 is achieved by running <pre>cryptsetup --allow-discards --persistent open /dev/nvme0n1p2 root</pre> or, if the device is already opened, via
       <pre>cryptsetup --allow-discards --persistent refresh root</pre> to write the flags persistently into the LUKS header: <br><br><pre>cryptsetup luksDump /dev/nvme0n1p2 | grep Flags</pre>

       > **Adding further flags overwrites any existing ones, so any preexisting flag needs to be respecified. On LUKS1 devices, this is somewhat more complicated and requires setting kernel
       parameters instead.**

2. Installation
   1. Select the mirror servers used to download packages. These are defined in `/etc/pacman.d/mirrorlist` and listed with descending priority. When the live system connects to the internet,
      the `reflector` command updates this list and sorts recently synchronized HTTPS mirrors by download rate. You may inspect the file to confirm satisfactory entries or edit accordingly.

      > **No other configuration except `/etc/pacman.d/mirrorlist` gets carried over from the live environment to the installed system.**

   3. Install essential packages to setup the system: <pre>pacstrap -K /mnt base linux linux-firmware</pre>

      > **The vanilla stable kernel `linux` is chosen over hardened `linux-hardened` or longterm `linux-lts` options. Similarly, due to only subtle differences, the `linux-zen` fork is not used.**

      Other important tools can be appended to the `pacstrap` command, although a later installation via `pacman` is also possible. Some suggestions include:
         1. Microcode updates for the common CPU brands, `amd-ucode` or `intel-ucode`
         2. Network manager software, `networkmanager`
         3. Console text editors, `nano` and `vim`

3. Configuration
   1. Thanks to the `udev` daemon, it is possible to have persistent block device names, which prevent many possible error sources. Nodes of drives sharing a naming scheme are added in arbitrary
      order on boot, so it is recommended to use the UUID by setting the corresponding `fstab` flag:
      
      <pre>genfstab -U /mnt >> /mnt/etc/fstab</pre>

      Check the resulting `/dev/etc/fstab` file and edit in case of errors.

      > **The contained static information about present filesystems will include the identifier, directory and type among other entries.**

   2. Change root into the new system to directly interact with its environment, tools and configurations: <pre>arch-chroot /mnt</pre>
   3. Set the time zone with <pre>ln -sf /usr/share/zoneinfo/<i>Region</i>/<i>City</i> /etc/localtime</pre> and generate `/etc/adjtime` by running:

      <pre>hwclock --systohc</pre>

      This assumes that the hardware clock is set to UTC standard time. To prevent clock drift, use an SNTP client:

      <pre>systemctl start systemd-timesyncd.service</pre>
  
      Outside the root user, starting and enabling the service is done as follows:
  
      <pre>timedatectl set-ntp true</pre>

      To check the service status or print verbose information, run:
  
      <pre>timedatectl status</pre>
      <pre>timedatectl timesync-status</pre>

      To adjust the provided time servers, edit `/etc/systemd/timesyncd.conf` or `/etc/systemd/timesyncd.conf.d/local.conf` and run <pre>timedatectl show-timesync --all</pre> to verify. Any
      servers from the [NTP Pool Project](https://www.ntppool.org/en/) can be used.

   4. Next, complete the localization. To use the correct region and language specific formatting, edit `/etc/locale.gen` and uncomment the locales you will be using, such as `de_DE.UTF-8 UTF-8`
      and `en_US.UTF-8 UTF-8` for German and English. Generate the locales:

      <pre>locale-gen</pre>

      Create the `/etc/locale.conf` file and set the `LANG=en_US.UTF-8` variable according to your language needs. If you changed the console keyboard layout before, enter `KEYMAP=de-latin1` in
      `/etc/vconsole.conf` to make it persistent. If desired, set the console font in `/etc/vconsole.conf` as well with the `FONT` and `FONT_MAP` variables.

   5. Configure the network for your system by first setting a consistent and identifiable hostname in `/etc/hostname` that conforms with the requirements. Next, connect to a network:

      <pre>nmcli device wifi connect <i>SSID</i> password <i>password</i></pre>
  
      To get a list of nearby networks and connections, or for activation and deletion, run:
  
      <pre>nmcli device wifi list</pre>
      <pre>nmcli connection show</pre>
      <pre>nmcli connection up <i>name</i></pre>
      <pre>nmcli connection delete <i>name</i></pre>

      TUI and GUI frontends are also available.

   6. Using the default initramfs based on BusyBox, additional hooks need to be added in the correct ordering due to using encryption. With these modifications, the `/etc/mkinitcpio.conf`
      should include something like this for its `HOOKS` variable:

      <pre>base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck</pre>

      > **The `encrypt` hook is required for encrypted root partitions and must be placed after `udev` is called. In early userspace, keyboards are enabled after `keyboard` has been run, and it
      can be helpful to place this before `autodetect` so that all drivers for frequently changing hardware are loaded. Overriding default settings is facilitated by the `keymap` and
      `consolefont` hooks.**

      Regenerate the initramfs:

      <pre>mkinitcpio -P</pre>

   7. Set a secure password for the root user to perform administrative actions with the `passwd` command.

      > **Local user information is stored as plain text `account:password:UID:GID:GECOS:directory:shell` in the `/etc/passwd` file. Using the `passwd` command, only a
      placeholder is used there to indicate the existence of a hashed passphrase in `/etc/shadow` with restricted access, making it much more secure and always recommended.**

   8. Install the `systemd-boot` boot manager that will load the kernel after startup. It is shipped with `systemd` which is a dependency of `base` itself. First make sure that the system is
      booted into UEFI mode with UEFI variables accessible. To do so, run

      <pre>efivar --list</pre>

      if it is installed, or

      <pre>ls /sys/firmware/efi/efivars</pre>

      and check whether the directory exists to proceed with the next steps. Use

      <pre>bootctl install</pre>

      to copy `systemd-boot` to the ESP as well as create a UEFI boot entry and set it as first in the UEFI boot order.

      > **Running `bootctl install` leads `systemd-boot` to search for the ESP in `/efi` and `/boot` per default.**

      Since there are issues with `bootctl install` while `arch-chroot` is active, newer versions should use the `--variables=yes` option: <pre>bootctl --variable=yes install</pre>

      Older systems can attempt the execution outside `arch-chroot` with `exit` and the `--esp-path=/mnt/boot` flag: <pre>bootctl --esp-path=/mnt/boot install</pre>
  
      Since this copies the EFI binaries from the ISO instead of the mounted root, odd incompatabilities might arise. To circumvent this, you can reenter `arch-chroot` and overwrite any
      preexisting EFI files, keeping only the boot entries created by the previous command:
  
      <pre>arch-chroot /mnt</pre>
      <pre>bootctl install</pre>

      As a last resort, create a manual boot entry with the pointer depending on bitness:

      <pre>efibootmgr --create --disk /dev/nvme0n1 --part 1 --loader '\EFI\systemd\systemd-bootx64.efi'<br>--label "Linux Boot Manager" --unicode</pre>
      <pre>efibootmgr --create --disk /dev/nvme0n1 --part 1 --loader '\EFI\systemd\systemd-bootia32.efi'<br>--label "Linux Boot Manager" --unicode</pre>

      > **Note that backslahes `\` are used instead of `/` as separators for the path**

   9. Adapt the boot loader configuration to include kernel parameters for encryption. Edit `/boot/loader/entries/arch.conf` and add

      <pre>cryptdevice=UUID=<i>device</i>:root root=/dev/mapper/root</pre>

      to the `options` line. Here, the device refers to the UUID of the LUKS superblock such as `nvme0n1p2` and must be replaced accordingly.

      > **The UUID in the likely preexisting `root` option should be replaced to avoid duplication conflicts, although in case of redundant entries, usually only the latter is registered
      by the kernel. Otherwise, the order does not matter as flags are independent from each another. Further parameters like `rw` or `ro` specify read write or read only mounting options.
      To show a splash screen and hide the boot process messages, both `splash` and `quiet` should be set.**

*Adapted from the [Arch Wiki](https://wiki.archlinux.org/)*.

<br>

### Supplementary Guide

1. For creating a bootable Microsoft Windows medium, first [download](https://www.microsoft.com/en-us/software-download) the appropriate installation image.
2. Verify the integrity by running
   
   <pre>sha256sum <i>path</i>/<i>to</i>/Win_<i>Generation_Version_Language</i>_x64.iso</pre>

   and comparing to the public hash list.

3. Plug in your USB flash drive, SSD or HDD, and determine its identifier. This is probably `sda` or some iteration like `sdb` and `sdc` on your system. Make sure you select the correct device by
   checking its properties, as otherwise critical data might be destroyed in the next step.
4. Wipe the drive and enter a `parted` console to create a GUID partition table. This will require root user privileges:
   <pre>wipefs -a /dev/sde</pre>
   <pre>parted /dev/sde</pre>
   1. <pre>mklabel gpt</pre>
   2. <pre>mkpart BOOT fat32 0% 1GiB</pre>
   3. <pre>mkpart INSTALL ntfs 1GiB 100%</pre>
   4. <pre>quit</pre>
   Instead of using the entire remainder of the drive, partition sizes as small as the ISO file are sufficient. Check the GPT layout:
   <pre>parted /dev/sde unit B print</pre>
5. Create the required directories, format the drive and copy data into the correct structure:
   1. Mount the ISO file,
      
      <pre>mkdir /mnt/iso</pre>
      <pre>mount <i>path</i>/<i>to</i>/Win_<i>Generation_Version_Language</i>_x64.iso /mnt/iso/</pre>

   2. Format the first partition of your drive as `FAT32` using `dosfstools` and mount it,
  
      <pre>mkfs.vfat -n BOOT /dev/sde1</pre>
      <pre>mkdir /mnt/vfat</pre>
      <pre>mount /dev/sde1 /mnt/vfat/</pre>

   3. Copy everything from the ISO except the `sources` directory here,
  
      <pre>rsync -r --progress --exclude sources --delete-before /mnt/iso/ /mnt/vfat/</pre>

   4. Copy only `boot.wim` from the `sources` directory while maintaining the same layout,

      <pre>mkdir /mnt/vfat/sources</pre>
      <pre>cp /mnt/iso/sources/boot.wim /mnt/vfat/sources/</pre>

   5. Format the second partition of your drive as `NTFS` using `ntfsprogs` and mount it,
  
      <pre>mkfs.ntfs --quick -L INSTALL /dev/sde2</pre>
      <pre>mkdir /mnt/ntfs</pre>
      <pre>mount /dev/sde2 /mnt/ntfs</pre>

   6. Copy everything from the ISO here,
  
      <pre>rsync -r --progress --delete-before /mnt/iso/ /mnt/ntfs/</pre>

6. Unmount the drive and synchronize to flush the buffer:

   <pre>umount /mnt/ntfs</pre>
   <pre>umount /mnt/vfat</pre>
   <pre>umount /mnt/iso</pre>
   <pre>sync</pre>

7. Power off the drive and remove it:

   <pre>udisksctl power-off -b /dev/sde</pre>

8. After booting the medium and beginning the installation process, you might want to proceed without internet connection and account creation. This way, drivers on external drives can be
   obtained more conveniently by directly executing the installer `.exe` instead of extracting `.inf` files from your device manufacturer. To achieve this, the menu screen propting for a
   network is modified after automatically rebooting by pressing `⇧` with `F10` and either `Fn` or `FnLock` before entering into the shell:

   <pre>oobe\bypassnro</pre>

*Adapted from the [NIXAID Blog](https://nixaid.com/)*.
