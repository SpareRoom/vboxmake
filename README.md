# virtualbox_base_box_builder

## Synopsis

This repo contains scripts and configuration files to automate the production of VirtualBox base boxes.  It's a work in progress, and has been developed specifically for use on OS X.  It is currently only suitable for installing CentOS 7.0, but it wouldn't require too much effort to install other Linux distros.

## The process

- The built-in VirtualBox PXE-enabled DHCP server supplies a new VM with an IP address, and specifies the path to the bootloader file.  This path is in the format `<vm name>.pxe`, so there must be one file per VM.

- Because VirtualBox uses [iPXE][1] firmware, we replace the standard `pxelinux.0` file with a simple [iPXE script][2] .  The script sets the TFTP prefix to the name of the VM, thus allowing separate PXE configurations to be created for different VMs.  The script then passes the location of the real `pxelinux.0` bootloader file to iPXE.  `pxelinux.0` is obtained from the [Syslinux PXELINUX][3] project.

- The PXE boot process loads `pxelinux.0`, which in turn loads the syslinux configuration menu file.  The config file provides the necessary kernel parameters, including the location of `initrd.img`, `vmlinuz` and the Kickstart configuration file.

- As part of the build process, an ISO file is generated that contains the Kickstart configuration file.  This ISO is attached to the VM as a secondary DVD-ROM drive so that the boot process can load it.  This negates the need to either build a custom `initrd.img` (which is rather difficult on OS X), or to have the file available over HTTP.

- Finally, the Kickstart configuration takes over, and installs the system.

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

The PXE boot loader installs a temporary root file system (`initrd.img`) and the Linux kernel (`vmlinuz`) into memory, and starts the kernel.  The Linux kernel is then able to continue with the boot process.  Both `initrd.img` and `vmlinuz` must be obtained from the distribution that is going to be installed.

These instructions are for the CentOS 7.0 minimal install ISO, but the commands for other versions and operating systems should be similar:

```
mkdir -p ~/tftpboot && cd $_
curl -O http://mirrors.melbourne.co.uk/sites/ftp.centos.org/centos/7.0.1406/isos/x86_64/CentOS-7.0-1406-x86_64-Minimal.iso

xorriso -osirrox on \
  -indev CentOS-7.0-1406-x86_64-Minimal.iso \
  -extract /images/pxeboot/ CentOS-7.0-1406-x86_64-Minimal/images/pxeboot/
```

### Download and extract Syslinux

I couldn't get VirtualBox PXE to work with syslinux 5.00+, but it works with 4.07:

```
mkdir -p ~/tftpboot && cd $_
curl -O https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-4.07.tar.gz

tar xvzf syslinux-4.07.tar.gz syslinux-4.07/core/pxelinux.0
```

## Notes

- The goal was for a solution that was quick to run, and which didn't require copying files to external HTTP or NFS servers.

- It looks as though it might be possible to bypass `pxelinux.0` altogether, and just use iPXE.  But the version of iPXE that is bundled with VirtualBox only has a minimal feature-set.  Attempts to [chain-load][6] a full version of iPXE seemed promising, but VirtualBox's DHCP server refused to issue a second IP address.

- Earlier revisions loaded the Kickstart configuration from a [local HTTP server][7], but in the end, creating a separate CD-ROM ISO proved simpler and more reliable.


[1]: http://ipxe.org/
[2]: http://ipxe.org/scripting
[3]: http://www.syslinux.org/wiki/index.php/PXELINUX
[4]: http://www.syslinux.org/wiki/index.php/Doc/isolinux#HYBRID_CD-ROM.2FHARD_DISK_MODE
[5]: http://www.gnu.org/software/xorriso/xorriso_eng.html
[6]: http://ipxe.org/howto/chainloading
[7]: http://www.andyjamesdavies.com/blog/javascript/simple-http-server-on-mac-os-x-in-seconds
