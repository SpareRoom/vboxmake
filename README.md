# virtualbox_base_box_builder

## Synopsis

This repo contains scripts and configuration files to automate the production of VirtualBox base boxes.

Notes:

The plan is to use VirtualBox's local PXE implementation to bootstrap the CentOS installation.  This requires a PXE boot loader file to be placed in `~/Library/VirtualBox/TFTP`.  It must be called `<vm name>.pxe`.  The SysLinux project's PXELINUX might do the trick:

http://www.syslinux.org/wiki/index.php/PXELINUX

## The process

- The built-in VirtualBox PXE-enabled DHCP server supplies a new VM with an IP address, and specifies the path to the bootloader file.  This path in the format `<vm name>.pxe`, so there must be one file per VM.

- Because VirtualBox uses [iPXE][1] firmware, we replace the standard `pxelinux.0` file with a simple [iPXE script][2] .  The script sets the TFTP prefix to the UUID of the VM, thus allowing different PXE menus to be created for different VMs.  The script then passes the location of the real `pxelinux.0` bootloader file to iPXE.  `pxelinux.0` is obtained from the [Syslinux PXELINUX][3] project.

- The PXE boot process loads `pxelinux.0`, and then looks for `initrd.img` and `vmlinuz` in the TFTP prefix directory (the VM UUID directory).  `initrd.img` creates the initial ramdisk, and `vmlinuz` is the Linux kernel.

- `pxelinux.0` looks for 

## Prerequisites

### Install xorriso

A number of Linux distributions are now supplied in a [hyrbid mode ISO format][4].  The OS X `hdiutil` command is unable to understand this format, and instead returns the error `no mountable filesystem`.  As an alternative, we'll use the [GNU xorriso][5] tool to extract the ISO file contents directly to disk.  I was going to use 7z, the command-line version of the 7-zip program, but unfortunately this does not support Rock Ridge filesystems, and therefore truncates some filenames to 64 characters.  This is a problem, as some of the files in the `repodata` directory are longer than this.

```
cd /tmp
curl -O http://www.gnu.org/software/xorriso/xorriso-1.3.9.tar.gz
tar xvzf xorriso-1.3.9.tar.gz
cd xorriso-1.3.9
./configure
make
sudo make install
```

### Download and extract CentOS

These instructions are for the CentOS 7.0 minimal install ISO, but the commands for other versions and operating systems should be similar:

```
mkdir -p ~/tftpboot && cd $_
curl -O http://mirrors.melbourne.co.uk/sites/ftp.centos.org/centos/7.0.1406/isos/x86_64/CentOS-7.0-1406-x86_64-Minimal.iso

mkdir CentOS-7.0-1406-x86_64-Minimal
xorriso -osirrox on -indev CentOS-7.0-1406-x86_64-Minimal.iso -extract / CentOS-7.0-1406-x86_64-Minimal
```

PXE booting requires these files to bootstrap the installation:

- `images/pxeboot/initrd.img`
- `images/pxeboot/vmlinuz`

### Download and extract Syslinux

I couldn't get VirtualBox PXE to work with syslinux 5.00+, but it works with 4.07:

```
mkdir -p ~/tftpboot && cd $_
curl -O https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-4.07.tar.gz

tar xvzf syslinux-4.07.tar.gz syslinux-4.07/core/pxelinux.0
```

### Syslinux

The PXE boot loader installs a temporary root file system (`initrd.img`) and the Linux kernel (`vmlinuz`) into memory, and starts the kernel.  The Linux kernel is then able to continue with the boot process.  Both `initrd.img` and `vmlinuz` can be found in the CentOS Linux distribution.


[1]: http://ipxe.org/
[2]: http://ipxe.org/scripting
[3]: http://www.syslinux.org/wiki/index.php/PXELINUX
[4]: http://www.syslinux.org/wiki/index.php/Doc/isolinux#HYBRID_CD-ROM.2FHARD_DISK_MODE
[5]: http://www.gnu.org/software/xorriso/xorriso_eng.html

