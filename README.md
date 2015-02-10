# sr_web_dev
Vagrant files and build scripts for the SpareRoom Web development environment

Notes:

The plan is to use VirtualBox's local PXE implementation to bootstrap the CentOS installation.  This requires a PXE boot loader file to be placed in `~/Library/VirtualBox/TFTP`.  It must be called `<vm name>.pxe`.  The SysLinux project's PXELINUX might do the trick:

http://www.syslinux.org/wiki/index.php/PXELINUX

I couldn't get VirtualBox PXE to work with syslinux 5.00+, but it works with 4.07:

```
curl -O https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-4.07.tar.gz
tar xvzf syslinux-4.07.tar.gz
syslinux-4.07/core/pxelinux.0 ~/Library/VirtualBox/TFTP/
```

Need to extract `vmlinuz` and `initrd.img` from distro.  

```
curl -L http://sourceforge.net/projects/p7zip/files/p7zip/9.38/p7zip_9.38_src_all.tar.bz2/download -o p7zip_9.38_src_all.tar.bz2
tar xvf p7zip_9.38_src_all.tar.bz2
cd p7zip_9.38
cp makefile.macosx_64bits makefile.machine
make 7z
bin/7z e \
  -o${HOME}/Library/VirtualBox/TFTP/ \
  ${HOME}/Downloads/CentOS-7.0-1406-x86_64-Minimal.iso \
  images/pxeboot/initrd.img images/pxeboot/vmlinuz

```


