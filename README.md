# NAME

vboxmake - Create VirtualBox VMs with unattended installation

# SYNOPSIS

    vboxmake \
      --linux-iso CentOS-7.0-1406-x86_64-Everything.iso \
      --unattended unattended/ks.cfg \
      --unattended unattended/custom.sh \
      --vm-name basebox-centos-7

    Options:

     -l, --linux-iso       ISO image from which to install Linux
     -i, --initrd          the path to initrd.img in `linux-iso`
     -k, --vmlinuz         the path to vmlinuz in `linux-iso`

     -s, --syslinux-targz  .tar.gz file from which to obtain pxelinux.0
     -p, --pxelinux0       the path to pxelinux.0 in `syslinux-targz`

     -u, --unattended      the path to the unattended installation script

     -n, --vm-name         the name to give to the VM
     -c, --vm-cpus         the number of CPUs to give to the VM
     -r, --vm-memory       the size of the VM RAM in MB
     -o, --vm-ostype       the VirtualBox OS type for this VM
         --vm-vram         the size of the VM VRAM in MB
     -d, --vm-hd-size      the size of the VM hard disk in MB

     -f, --force           overwrite existing VM of same name

     -t, --tftp-dir        the path to VirtualBox's TFTP directory
     -v, --vboxmanage      the path to the VBoxManage executable
     -g, --vboxguest       the path to the VBoxGuestAdditions.iso file
     -x, --xorriso         the path to the xorriso executable
         --xorrisofs       the path to the xorrisofs executable

     -h, --help            brief help message
     -m, --man             full documentation

# DESCRIPTION

`vboxmake` is a Perl script designed to help automate the production of
VirtualBox virtual machines.

**ALPHA CODE ALERT:**  This script is a work in progress, and has been developed
and tested only on OS X Yosemite.  It's interface and functionality will
almost certainly change.  It is currently only suitable for installing
CentOS 7.0, but it probably wouldn't require much effort to make it install
other Linux distros.  Patches and pull requests are welcome...

## The build process

- The built-in VirtualBox PXE-enabled DHCP server supplies a new VM with an
IP address, and specifies the path to the bootloader file.  This path is in
the format `<vm name>.pxe`, so there must be one file per VM.
- Because VirtualBox uses [iPXE](http://ipxe.org/) firmware, we replace the
standard `pxelinux.0` file with an [iPXE script](http://ipxe.org/scripting).
The script sets the TFTP prefix to the name of the VM, thus allowing separate
PXE configurations to be created for different VMs.  The script then passes
the location of the real `pxelinux.0` bootloader file to iPXE.
`pxelinux.0` is obtained from the [Syslinux PXELINUX](http://www.syslinux.org/wiki/index.php/PXELINUX)
project.
- The PXE boot process loads `pxelinux.0`, which in turn loads the syslinux
configuration menu file.  The config file provides the necessary kernel
parameters, including the location of `initrd.img`, `vmlinuz` and the
Kickstart configuration file.
- As part of the build process, an ISO file is generated that contains the
Kickstart configuration file.  This ISO is attached to the VM as a secondary
DVD-ROM drive so that the boot process can load it.  This negates the need
to either build a custom `initrd.img` (which is rather difficult on OS X),
or to have the file available over HTTP.
- Finally, the Kickstart configuration takes over, and installs the system.

# OPTIONS

- -l, --linux-iso

    **Required.**

    `vboxmake` will attempt to extract `initrd.img` and `vmlinuz` from this ISO
    image.  It will also attach this ISO image to the IDE storage controller for
    use as the installation media.

- -i, --initrd

    Default: `/images/pxeboot/initrd.img`

    This is the location of the `initrd.img` file in the Linux ISO.

- -k, --vmlinuz

    Default: `/images/pxeboot/vmlinuz`

    This is the location of the `vmlinuz` file in the Linux ISO.

- -s, --syslinux-targz

    Default: `${HOME}/vboxmake/syslinux-4.07.tar.gz`

    `vboxmake` will attempt to extract `pxelinux.0` from this .tar.gz file.
    It is highly recommended that Syslinux version 4.0.7 is used, as I have
    been unable to get later versions working correctly with VirtualBox.

- -p, --pxelinux0

    Default: `syslinux-4.07/core/pxelinux.0`

    This is the location of `pxelinux.0` in the Syslinux .tar.gz file.

- -u, --unattended

    **Required.**

    This is the unattended installation file that will be used to script the
    installation of your VM.  This option may be specified multiple times:
    each script will be added to the **root** of the unattended ISO file, and
    the first file specified will be used as the initial configuration file.
    This allows configuration files to include other configuration files.

- -n, --vm-name

    **Required.**

    Passed to `VBoxManage createvm --name`.

- -c, --vm-cpus

    Default: `2`

    Passed to `VBoxManage modifyvm --cpus`

- -r, --vm-memory

    Default: `1024`

    Passed to `VBoxManage modifyvm --memory`

- -o, --vm-ostype

    Default: `RedHat_64`

    Passed to `VBoxManage modifyvm --ostype`

- --vm-vram

    Default: `10`

    Passed to `VBoxManage modifyvm --vram`

- -d, --vm-hd-size

    Default: `28610`

    Passed to `VBoxManage createhd --size`

- -f, --force

    If `vboxmake` detects a VirtualBox VM with the same name as `--vm-name`,
    it will abort.  Specifying `--force` will cause the VM and any associated
    configuration files to be overwritten.

- -t, --tftp-dir

    Default: `${HOME}/Library/VirtualBox/TFTP`

    When the first NIC is of type NAT, VirtualBox will look in the TFTP directory
    for a file with the name `--vm-name` and the extension `.pxe`.  For example,
    if the VM name is `basebox`, the TFTP boot process would look for a file
    called `basebox.pxe` in the TFTP directory.  `vboxmake` creates the
    directory and the file automatically.  You should only need to change this
    setting if your installation of VirtualBox is using a different location
    for the TFTP directory.

- -v, --vboxmanage

    Default: `/usr/bin/VBoxManage`

    This is the path to the `VBoxManage` executable.

- -g, --vboxguest

    Default: `/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso`

    This is the path to the `VBoxGuestAdditions.iso` file from which
    `VBoxLinuxAdditions.run` will be extracted.  The extracted file will then
    be added to `unattended.iso` under the `/vboxguest` directory.

- -x, --xorriso

    Default: `/usr/local/bin/xorriso`

    This is the path to the `xorriso` executable.

- --xorrisofs

    Default: `/usr/local/bin/xorrisofs`

    This is the path to the `xorrisofs` executable.

# PREREQUISITES

## xorriso

xorriso is used to extract `initrd.img` and `vmlinuz` from the Linux
distribution, and to create the ISO CD-ROM image that contains the Kickstart
configuration file.  Installation is easy:

    cd /tmp
    curl -O http://www.gnu.org/software/xorriso/xorriso-1.3.9.tar.gz

    tar xvzf xorriso-1.3.9.tar.gz
    cd xorriso-1.3.9

    ./configure
    make
    sudo make install

## Linux installation ISO

`vboxmake` has so far only been tested with CentOS 7.0 minimal.

Download a copy of the installation ISO, and place it somewhere handy.
There's no need to manually extract anything from the ISO.

## Syslinux

`vboxmake` needs a copy of `pxelinux.0` from the Syslinux PXELINUX project.
It is highly recommended that Syslinux version 4.0.7 is used, as I have
been unable to get later versions working correctly with VirtualBox.
`vboxmake` will attempt to extract `pxelinux.0` from this .tar.gz file.

# QUICK START

This quick start guide will take you through the steps necessary to build a
minimal CentOS 7.0 VirtualBox VM, from which you will then create a Vagrant
base box.

- vboxmake directory

    Create a `vboxmake` directory in which to put some downloads:

        mkdir -p ~/vboxmake

- CentOS

    Download a copy of [CentOS-7.0-1406-x86\_64-Everything.iso](http://isoredirect.centos.org/centos/7/isos/x86_64/), and put it in `~/vboxmake`.

- Syslinux

    Download a copy of [Syslinux 4.07](https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-4.07.tar.gz), and put it in `~/vboxmake`.

- xorriso

    Download and install [xorriso](http://www.gnu.org/software/xorriso/#download).  See [PREREQUISITES](https://metacpan.org/pod/PREREQUISITES).

- VirtualBox

    Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

- Vagrant

    Download and install [Vagrant](http://www.vagrantup.com/downloads.html).

- Create a base box

        bin/vboxmake \
          --linux-iso ~/vboxmake/CentOS-7.0-1406-x86_64-Everything.iso \
          --unattended=unattended/ks.cfg \
          --unattended=unattended/custom.sh \
          --force \
          --vm-name "basebox-centos-7"

    Wait for the VM to build and then shutdown.

- Make a Vagrant base box for VirtualBox

        mkdir ~/vagrant-centos-7 && cd $_
        vagrant package --base basebox-centos-7
        vagrant box add centos-7-box package.box
        vagrant init centos-7-box
        vagrant up
        vagrant ssh

# NOTES

To update the README from this POD, install [Pod::Markdown](https://metacpan.org/pod/Pod::Markdown), and then:

    perldoc -Tu bin/vboxmake | pod2markdown > README.md
