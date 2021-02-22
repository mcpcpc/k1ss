INSTALLATION
============

Welcome to the installation guide for KISS Linux.

The installation utilizes a tarball which is unpacked to '/'. This same tarball
is also directly usable as a chroot from any Linux distribution.

A live-CD from a another distribution is required to bootstrap KISS. It does not
matter which distribution is used so long as it includes 'tar' and other basic
utilities.

From this point on, the guide assumes you have booted a live-CD, are logged
in as root, have your disks, partitions and filesystems setup and have an
internet connection.

[2.0] Install KISS
------------------

NOTE: Disks should be setup and fully mounted to /mnt.

Start by declaring a variable.

    $ url={{ site.kiss.git }}/repo/releases/download/2020.9-2  

[2.1] Download the latest release
---------------------------------

    $ wget "$url/kiss-chroot-2020.9-2.tar.xz"

[2.2] Verify the checksums (recommended)
----------------------------------------

This step verifies that the release matches the checksums generated uponits 
creation and also ensures that the download completed successfully.

    $ wget "$url/kiss-chroot-2020.9-2.tar.xz.sha256" 
    $ sha256sum -c < kiss-chroot-2020.9-2.tar.xz.sha256 

[2.3] Verify the signature (recommended)
----------------------------------------

This step verifies that the release was signed by its creator, Dylan Araps. If
the live OS of your choice does not include GPG, this step can also be done on
another machine (with the same tarball).

Download the armored ASCII file:

    $ wget "$url/kiss-chroot-2020.9-2.tar.xz.asc"

Import my public key (if this fails, try another keyserver):

    $ gpg --keyserver keys.gnupg.net --recv-key 46D62DD9F1DE636E 

Verify the signature:

    $ gpg --verify "kiss-chroot-2020.9-2.tar.xz.asc" 

[2.4] Unpack the tarball
------------------------

This step effectively installs KISS to /mnt. The tarball contains a full system 
minus the bootloader, kernel and optional utilities.

    $ cd /mnt
    $ tar xvf /path/to/kiss-chroot-2020.9-2.tar.xz

[2.5] Enter the chroot
----------------------

This is a simple script to chroot into /mnt and set up the environment for the
rest of the installation. The script handles mounting pseudo filesystems (/proc, 
/dev, /sys), enabling network inside the chroot, etc.

On execution of this step you will be running KISS! The next steps involve the 
kernel, software compilation and system setup.

    $ /mnt/bin/kiss-chroot /mnt 

[3.0] Setup repositories
------------------------

The repository system is quite different to that of other distributions. The 
system is controlled via an environment variable called KISS_PATH. This variable
is analogous to PATH, a colon separated list of absolute paths.

A repository is merely a directory (repo) containing directories (packages) and
can be located anywhere on the file-system. The full path to the directory is
the value to KISS_PATH.

The chroot/installation tarball does not come with any repositories by default,
nor does the package manager expect or /assume/ that any exist in a given 
location. This is entirely up to the user.

[3.1] Setting KISS_PATH
-----------------------

The variable can be set system-wide, per-user, per-session, per-command, and 
even programmatically. This guide will cover setting it for the current user 
with an example repository layout.

Take this layout:

    ~/repos/
      |
      +- repo/
      |  - .git/
      |  - core/
      |  - extra/
      |  - xorg/
      |
      +- community/
      |  - .git/
      |  - community/
      |
      +- personal/
      |  - games/
      |  - web
      |     

*   Repositories are stored in ~/repos/ which is a per-user configuration.
*   There are two git repositories containing four KISS repositories.
*   There is a third repository called personal, not tracked by git.
*   The personal repository contains two KISS repositories.

This user's KISS_PATH could look like this:

    $ cat ~/.profile
    ...
    export KISS_PATH=''
    KISS_PATH=$KISS_PATH:$HOME/repos/personal/games
    KISS_PATH=$KISS_PATH:$HOME/repos/personal/web
    KISS_PATH=$KISS_PATH:$HOME/repos/repo/core 
    KISS_PATH=$KISS_PATH:$HOME/repos/repo/extra
    KISS_PATH=$KISS_PATH:$HOME/repos/repo/xorg
    KISS_PATH=$KISS_PATH:$HOME/repos/community/community
    ...

*   All repositories are enabled.
*   This is a per-user configuration using ~/.profile
*   The package manager will search this list in the order it is defined.

> TIP: Run '. ~/.profile' for changes to immediately take effect.
> TIP: Run 'echo "$KISS_PATH"' to check if properly set.

[3.2] Official repositories
---------------------------

The official repositories contain everything from the base system to a working
web browser (Firefox) and media player (mpv). This includes Xorg, rust, nodejs
and a lot of other useful software.

Clone the repository to the directory of your choosing.

    $ git clone {{ site.kiss.git }}/repo 

This will be cloned to a directory called 'repo'. This directory contains
multiple KISS repositories (core, extra, testing and xorg). Core and Extra must
be enabled as this guide requires their use. Xorg is optional.

[3.3] Community
---------------

The community repository contains packages submitted and maintained by users of
the distribution. It is twice the size of the official repositories and contains
a lot of useful software.

Clone the repository to the directory of your choosing.

    $ git clone {{ site.kiss.git }}/community

This will be cloned to a directory called 'community'. This directory contains a 
single KISS repository bearing the same name.

[3.4] Universe
--------------

There are many more repositories in existence, each providing a unique set of
software. These are all independently created and managed by users. This has
been called the "KISS Universe".

*   <{{ site.kiss.web }}/wiki/community/repositories>
*   <{{ site.kiss.gh }}/topics/kiss-repo>

NOTE: It may be desirable to save this step for post-installation.

Repositories should now be setup and in functioning order. Run 'kiss search \*' 
(notice the escaping) to print all repository packages in the search order of
the package manager.

[4.0] Enable repository signing
-------------------------------

This step is optional and can also be done after the installation. Repository 
signing ensures that all updates have been signed by (Dylan Araps) and further 
prevents any unsigned updates from reaching your system.

[4.1] Build and install gnupg1
------------------------------

Welcome to the KISS package manager! These two commands are how individual
packages are built and installed on a KISS system.

    $ kiss build   gnupg1
    $ kiss install gnupg1

[4.2] Import my (Dylan Araps') key
----------------------------------

If the GNU keyserver fails on your network, try an alternative mirror.

Import my public key:

    $ gpg --keyserver keys.gnupg.net --recv-key 46D62DD9F1DE636E
    
Trust my public key:

    $ echo trusted-key 0x46d62dd9f1de636e >>/root/.gnupg/gpg.conf 

[4.3] Enable signature verification
-----------------------------------

Repository signature verification uses a feature built into git itself 
(merge.verifySignatures)!  This can be disabled at any time using the inverse of
the below git command.

The same steps can also be followed with 3rd-party repositories if the owner
chooses to sign their commits.

    $ cd /var/db/kiss/repo
    $ git config merge.verifySignatures true

[5.0] Rebuild kiss
------------------

This step is entirely optional and can also be done post-installation.

[5.1] Modify compiler options (optional)
----------------------------------------

These options have been tested and work with every package in the repositories.
If you'd like to play it safe, use -O2 or -Os instead of -O3.

If your system has a low amount of memory, omit -pipe. This option speeds up 
compilation but may use more memory.

If you intend to transfer packages between machines, omit -march=native. This
option tells the compiler to use optimizations unique to your processor's 
architecture.c

The `-jX` option should match the number of CPU threads available. You can omit 
this, however builds will then be limited to a single thread.

CFLAGS/CXXFLAGS:

    # NOTE: The 'O' in '-O3' is the letter O and NOT 0 (ZERO). 
    $ export CFLAGS="-O3 -pipe -march=native"
    $ export CXXFLAGS="$CFLAGS"

MAKEFLAGS:

    # NOTE: 4 should be changed to match the number of threads.
    $ export MAKEFLAGS="-j4"

[5.2] Update all base packages to the latest version
----------------------------------------------------

This is how updates are performed on a KISS system. This command uses git to
pull down changes from all enabled repositories and will then optionally handle
the build/install process.

    $ kiss update

[5.3] Rebuild all packages
--------------------------

We simply cd to the installed packages database and use a glob to grab the name
of every installed package. This glob is then passed to the package manager as a 
list of packages to build.

    $ cd /var/db/kiss/installed && kiss build *

[6.0] Userspace tools
---------------------

Each kiss action (build, install, etc) has a shorthand alias. From now on, 'kiss
b' and 'kiss i' will be used in place of 'kiss build' and 'kiss install'.

The software below is required unless stated otherwise.

[6.1] Filesystem utilities
--------------------------

NOTE: Open an issue for additional filesystem support.

EXT2, EXT3, EXT4:

    $ kiss b e2fsprogs
    $ kiss i e2fsprogs

FAT, VFAT:

    $ kiss b dosfstools 
    $ kiss i dosfstools

XFS:

    $ kiss b xfsprogs
    $ kiss i xfsprogs

[6.2] Device management
-----------------------

NOTE: If you choose to not install eudev, mdev will automatically be used in its 
place. Eudev is recommended as a lot of software requires it. See: 
<{{ site.kiss.web }}/wiki/dev/replacing-udev> for more information.

Needed for blkid support in eudev (recommended but not required):

    $ kiss b util-linux 
    $ kiss i util-linux

The device manager:

    $ kiss b eudev 
    $ kiss i eudev

[6.3] WiFi (optional)
---------------------

    $ kiss b wpa_supplicant
    $ kiss i wpa_supplicant 

[6.4] Dynamic IP addressing (optional)
--------------------------------------

    $ kiss b dhcpcd
    $ kiss i dhcpcd

[7.0] The hostname
------------------

Create the /etc/hostname file:

    $ echo HOSTNAME > /etc/hostname

Update the /etc/hosts file:

    127.0.0.1  HOSTNAME.localdomain  HOSTNAME
    ::1        HOSTNAME.localdomain  HOSTNAME  ip6-localhost

NOTE: This step must be done every time the hostname is changed.

[8.0] The kernel
----------------

This step involves configuring and building your own Linux kernel. If you have
not done this before, below are a few guides to get you started.

*   <https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide>
*   <https://wiki.gentoo.org/wiki/Kernel/Configuration>
*   <https://kernelnewbies.org/KernelBuild>

The Linux kernel is not managed by the package manager. The kernel is managed 
manually by the user. (Rationale: <{{ site.kiss.web }}/wiki/kiss/faq#7.3)>

KISS does not support booting using an initramfs (see <{{ site.kiss.git }}/wiki/faq#7.2>). When
configuring your kernel ensure that all required file-system, disk controller
and USB drivers are built with [*] (=y) and not [m] (=m).

A patch may be required for some kernels when built with GCC 10.1.0. Please read
the link (and the patch itself for more information). @/news/20200509a

TIP: The Wiki contains a basic kernel configuration page. <{{ site.kiss.web }}/wiki/kernel/config>

[8.1] Install required packages
-------------------------------

    # libelf (required in most if not all cases). 
    $ kiss b libelf
    $ kiss i libelf

    # ncurses (required only for 'make menuconfig'). 
    $ kiss b ncurses
    $ kiss i ncurses

    # perl (required in nearly all cases).
    $ kiss b perl
    $ kiss i perl

TIP: A patch can be applied to remove this requirement.
*   <{{ site.kiss.web }}/wiki/kernel/config#5.0>
*   /usr/share/doc/kiss/wiki/kernel/patches/kernel-no-perl.patch

[8.2] Download the kernel sources
---------------------------------

Kernel releases:

*   <https://kernel.org> (vanilla)
*   <https://www.fsfla.org> (libre)

A larger list of kernels can be found here:
<https://wiki.archlinux.org/index.php/Kernel>

Download the kernel sources:

    $ wget KERNEL_SOURCE

Extract the kernel sources:

    $ tar xvf KERNEL_SOURCE 
    $ cd linux-*

[8.3] Download firmware blobs (if required)
-------------------------------------------

To keep the KISS repositories entirely FOSS, the proprietary kernel firmware is 
omitted. This also makes sense as the kernel itself is manually managedv by the 
user. This step is only required if your hardware utilizes makes useb of this 
firmware.

<https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git>

Download and extract the firmware:

    $ wget FIRMWARE_SOURCE.tar.gz
    $ tar xvf linux-firmware-20191022.tar.gz

Copy the required drivers to '/usr/lib/firmware':

    $ mkdir -p /usr/lib/firmware
    $ cp -R ./path/to/driver /usr/lib/firmware

[8.4] Configure the kernel
--------------------------

You can determine which drivers you need by searching the web for your hardware
and the Linux kernel.

If you require firmware blobs, the drivers you enable must be enabled as [m]
(modules). You can also optionally include the firmware in the kernel itself.

Generate a default config with most drivers built into the kernel:

    $ make defconfig

Open an interactive menu to edit the generated .config and enable anything extra
you may need:

    $ make menuconfig

Store the generated config for reuse later:

    $ cp .config /path/to/somewhere

TIP: The kernel can backup its own .config file. <{{ site.kiss.web }}/wiki/kernel/config#2.0>

[8.5] Build the kernel
----------------------

This may take a while to complete. The compilation time depends on your hardware
and kernel configuration. The 'nproc' command outputs the total number of
threads which we pass to make for a multi-threaded build.

    $ make -j "$(nproc)" 

[8.6] Install the kernel
------------------------

Install the built modules (to /usr/lib):

    $ make INSTALL_MOD_STRIP=1 modules_install

Install the built kernel (to /boot). (Ignore the LILO error):

    $ make install

Rename the kernel/system.map (vmlinuz -> vmlinuz-VERSION):

    $ mv /boot/vmlinuz    /boot/vmlinuz-VERSION
    $ mv /boot/System.map /boot/System.map-VERSION

[9.0] The bootloader
--------------------

The default bootloader is grub (though nothing prevents the use of another 
bootloader). This default was chosen as most people are familiar with it, both
BIOS and UEFI are supported and vast amounts of documentation for it exists.

[9.1] Recommended
-----------------

Have an /etc/fstab file ready.

[9.2] Build and install grub
----------------------------

    $ kiss b grub
    $ kiss i grub 

Required for UEFI:

    $ kiss b efibootmgr
    $ kiss i efibootmgr

[9.3] Setup grub
----------------

BIOS:

    $ grub-install /dev/sdX
    $ grub-mkconfig -o /boot/grub/grub.cfg

UEFI (replace 'esp' with the EFI mount point):

    $ grub-install --target=x86_64-efi \
                   --efi-directory=esp \
                   --bootloader-id=kiss
    $ grub-mkconfig -o /boot/grub/grub.cfg

[10.0] Install init scripts
---------------------------

The default init is busybox init (though nothing ties you to it). The below
commands install the bootup and shutdown scripts as well as the default inittab 
config.

Source code: <{{ site.kiss.git }}/init>

    $ kiss b baseinit
    $ kiss i baseinit

[11.0] Change the root password (recommended)
---------------------------------------------

    $ passwd root

[12.0] Add a normal user (recommended)
--------------------------------------

    $ adduser USERNAME

[13.0] Install Xorg (optional)
------------------------------

To install Xorg, the input drivers and a basic default set of fonts, run the 
following commands. See <{{ site.kiss.web }}/wiki/xorg>

    $ kiss b xorg-server xinit xf86-input-libinput

Installing a base font is recommended as nothing will work without fonts:

    $ kiss b liberation-fonts
    $ kiss i liberation-fonts

[13.1] Add your user to the relevant groups
-------------------------------------------

This groups based permissions model may not be suitable if KISS will be used as
a multi-seat system. Further configuration can be done at your own discretion.

Replace 'USERNAME' with the name of the user created earlier:

    $ addgroup USERNAME video
    $ addgroup USERNAME audio

[14.0] Further steps
--------------------

You should now be able to reboot into your KISS installation. Typical
configuration should follow (creation of users, service configuration, 
installing a window manager, terminal etc).

The KISS Wiki is a good place to look for post-installation information.

TIP:

    # The Wiki is available offline via 'kiss help wiki'.
    
    $ kiss help wiki

    $ kiss help wiki/xorg
    $ kiss help wiki/xorg/xinit
    $ kiss help wiki/xorg/xinit

    $ kiss help wiki/software
    $ kiss help wiki/software/man-pages

If you encountered any issues, don't hesitate to open an issue on one of our 
GitHub repositories, post on <https://reddit.com/r/kisslinux> or join the
IRC server.

See: <{{ "/" | absolute_url }}contact>
