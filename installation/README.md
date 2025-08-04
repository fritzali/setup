## Installation

In the following, steps for installing an [Arch Linux](https://archlinux.org/) based system with a simple encrypted partition scheme are listed. To get the most recent firmware versions, it might be useful to
first install [Microsoft Windows](https://www.microsoft.com/en-us/windows) to download any available updates and activate your hardware.

#### Main Guide

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


2. Installation
3. Configuration

*Adapted from the [Arch Wiki](https://wiki.archlinux.org/)*.

#### Supplementary Guide

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

*Adapted from the [NIXAID Blog](https://nixaid.com/)*.
