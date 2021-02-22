FREQUENTLY ASKED QUESTIONS
==========================

May you find what you seek.

[1.0] Why should I use KISS?
----------------------------

That's up to you to decide.

I'm not going to try and sell you something I'm giving away for free. Have a
read of the website and its wealth of information, try KISS in a virtual
machine (a chroot is also an option) and come to your own conclusions.

[2.0] What does KISS mean?
--------------------------

KISS is an acronym for "Keep it simple stupid" (notice _no_ comma).

Stupid does not refer to the user. To quote Wikipedia:

> The principle is best exemplified by the story of Johnson handing a team of
  design engineers a handful of tools, with the challenge that the jet aircraft
  they were designing must be repairable by an average mechanic in the field
  under combat conditions with only these tools.

> Hence, the "stupid" refers to the relationship between the way things break
  and the sophistication available to repair them.

[3.0] Is KISS rolling release or stable?
----------------------------------------

Packages are updated as quickly as possible while at the same time tested to
ensure that no issues arise in the new version. A new version of a package will
be held back if it causes issues.

When a package update brings in a bug during the build process or at runtime,
it will be patched in KISS until it is fixed upstream. This will typically
happen hours after the version is available and doesn't cause a delay.

Nothing prevents you as a user from choosing the update schedule you would
prefer. Total control is in your hands. KISS merely provides you with an always
up-to-date repository pool.

[4.0] Something something BUS factor of ONE.
--------------------------------------------

Every user of the distribution owns their system in its entirety. The management
of the distribution which extends to the management of updates to the user's
system is entirely optional.

All KISS systems contain the full sources for the distribution and each user
has the means of managing and maintaining their machine on their lonesome (or
by forming a collective and secondary "upstream").

This includes:

*   The repositories with full git history.

    The repository updates are simply a 'git pull' which results in each user
    having a full copy of the git repositories on their machine at /var/db/kiss/.

*   The package manager and the kiss-* scripts.

    As these are all simple shell scripts, the installed "binaries" _are_ the
    unchanged source code. All of these are self-contained and separate programs.

    The 'kiss-outdated' script allows one to check their system for outdated
    packages against the repology.org database independent of whether or not
    repology has support for KISS itself.

    The 'kiss-export' script allows one to convert an installed package back into
    a redistributable and installable binary tarball. Simply run
    'kiss-export pkg_name' and a tarball will be created in the current directory.

*   The distribution's documentation.

    As the website sources are merely plain .txt files, the docs are available
    in /usr/share/doc/kiss and are readable in less, vim or the program of your
    choosing.

*   The init scripts.

    In addition to the reasoning given for the package manager above, KISS has no
    lock-in regarding how the machine boots. Were the init scripts to disappear,
    there would be no real loss.

    To continue further, the init scripts need not change. There is no need to
    update them. Any extensions to them can be made via the /etc/rc.d directory
    or the /etc/rc.conf file.

What all of this means is that were the GitHub or website to go down, there
would be no loss in code, documentation or the distribution itself.

It also extends to each user having everything they will need to continue the
distribution for themselves or for other users. A change of git remote is all
that is needed in the latter case.

KISS was designed to be maintainable by a single person. The repositories are
kept small and focused. There is no requirement or _need_ for infrastructure of
any kind.

A user today could choose to go their own way with their system. Everything
they need is already in the existing system. The power is in your hands.
You are free.

[5.0] SOFTWARE
--------------

[5.1] Why isn't SOFTWARE packaged?
----------------------------------

No one has packaged it yet.

[5.2] Can you package SOFTWARE for me?
--------------------------------------

Do it yourself.

[5.3] What init does KISS use?
------------------------------

KISS uses busybox init by default. There is however, no lock-in and the user may
use whatever they like. The distribution's boot up scripts are written in an
init-agnostic way and act as a base for every init to use.

[5.4] What coreutils does KISS use?
-----------------------------------

KISS uses busybox coreutils by default. There is however, no lock-in and the user
may use whatever they like. All shell code is written in portable POSIX shell and
coreutils invocations.

*   One exception is made for 'sed -i' which isn't POSIX but is far too useful
    to do away with. Many sed implementations include -i so this is more of a
    non-issue.

*   The tar command has no standard interface, however support is included for
    busybox tar, libarchive tar (bsdtar), GNU tar and others.

*   Some utilities have no standard specification and where their use is
    required, we adhere to common options between implementations.

*   Barring the above exceptions, users are running KISS without the use of
    busybox. Popular alternatives are Suckless' sbase/ubase and GNU's set of
    core utilities.

[5.5] Does DRM work in browsers?
--------------------------------

DRM (specifically the commonly used Widevine) does not work when musl is the C
library. The DRM is a binary (typically downloaded by the browser) and it
requires glibc.

Nothing can be done to fix this as the DRM is not open source.

Workarounds:

*   Running your browser inside a chroot (containing a glibc based
    distribution).

*   Running the Flatpak (available here $/dylanaraps/kiss-flatpak) version of
    the browser. NOTE: Requires pulseaudio ($/periish/kiss-all)

*   Using something such as Nix to run the browser.

*   Running the browser in docker, a virtual machine, etc.

[6.0] Package Management
------------------------

[6.1] How do I remove a package and all of its dependencies?
------------------------------------------------------------

Short answer: With one command? You can't.

The package manager does not do recursive dependency removal on removal of a
package. This error-prone automation will not be added to the package
manager.

Instead, the workflow is to remove the single package and then look at the
output of the 'kiss-orphans' command to see what can now be removed. This
command will list all packages which have no relationship with other packages,
otherwise known as orphans.

This list may include Firefox and other "end" software so a brain is required
when parsing the list. You'll come to learn the relationships between packages
and their dependencies and this will eventually become effortless.

[7.0] Kernel
------------

[7.1] Why doesn't my kernel boot?
---------------------------------

The kernel not booting can be a variety of issues. This is almost always related
to a configuration issue in the kernel, /etc/fstab or the bootloader.

KISS doesn't use an initramfs so the configuration of the kernel may have
different requirements to other distributions.

1.  The drivers for your disk controller, drives and filesystems must not be
    built as kernel modules. They should be set to =y in your .config or [*]
    when using make menuconfig.

    Essentially, every driver the kernel requires to detect and mount the
    drive containing the root filesystem, must be built as a part of the kernel
    binary.

2.  Multi-drive systems must use PARTUUID or UUID in place of /dev/sdXX in the
    bootloader configuration to ensure that the kernel will find the right
    drive.

[7.2] Why doesn't KISS support initramfs?
-----------------------------------------

KISS technically supports booting via an initramfs, it just doesn't require or
provide one. As a user you have the means to set this up yourself for your
system.

Full disk encryption is also possible without the use of an initramfs in modern
kernels (see dm-mod.create).

The initramfs concept is an ugly, complicated and largely optional mess. Thank
god it isn't a requirement.

[7.3] Why must I compile my own kernel?
---------------------------------------

The kernel must be compiled by the user for a variety of reasons.

1.  The user maintains full control over all aspects of their kernel and
    further, their entire system. There is no lock-in into a specific set of
    kernel sources, version, use of proprietary firmware, patches, config, etc.

    The user decides:

    *   Which kernel to use (sources and version).
    *   When to update their kernel.
    *   Whether to use proprietary firmware blobs.
    *   Which patches to use (if any).
    *   How the kernel is configured (endless options).
    *   How many kernels they'd like to keep.
    *   Compiler options (-O3, -march=native, etc).

    You as a user might actually learn something too. You may come to understand
    your hardware, what drivers it needs, how the kernel works from a
    configuration perspective, etc.

    You should have a better understanding of this part of the system afterwards
    and you'll be able fix any issues at this level with relative ease.

    Remember, it's only hard the first time. Once a working config is created,
    no further work should need to be done each time you update your kernel.

    If the new kernel version has an issue with your hardware, simply boot
    another from your backups. If new hardware was added, simply run 'make
    menuconfig' and add what is needed.

    I'll say what I always say. Nothing prevents anyone from providing kernel
    binaries and an initramfs generation tool for KISS. Just don't wait on the
    BDFL to do it for you.

2.  Eases distribution maintenance for the BDFL. Shipping a generic kernel
    demands a humongous, module heavy kernel with support for everything under
    the sun. An initramfs is then a requirement to boot this damn monstrosity.

    There is then a need for a _portable_ initramfs generation tool which to my
    knowledge doesn't exist. There can't be a dependence on bash or anything
    outside of core. All initramfs tools are either distribution-specific or
    non-portable to KISS.

    Then there'd be endless support requests for tweaks, additions and removals
    to the distribution config and the burden of updating the kernel on every
    release.

    Does KISS ship the latest kernel? The long term support kernel? Both? Some
    users require firmware so we'd need two separate binaries, one for
    linux-libre and another for regular linux.

    What if the latest kernel has issues on some hardware? New builds and
    binaries must be released with backported patches.

    It's a large maintenance burden for something which can simply be solved by
    the user doing this themselves. The user maintains full control over every
    aspect of their kernel and is solely responsible for it.

[7.4] Why doesn't KISS provide kernel sources as a package?
-----------------------------------------------------------

Why should it? See above.

[7.5] Must I keep the linux-headers package in sync with my kernel's
      version to maintain a working system?
-------------------------------------------

The kernel headers in KISS are pinned to an LTS kernel version and are only
updated when there are changes of interest in the kernel or headers themselves
(usually by users requesting new features available in the newer headers).

The headers are backwards compatible and are fully usable with a matching or
NEWER kernel version. There are two cases where you'd be required to update the
headers yourself.

1.  To run a kernel _older_ than the default headers.
2.  To make use of features in your _newer_ kernel version.

Here's an excerpt from the kernel's documentation which may explain the
situation better than I.

> Kernel headers are backwards compatible, but not forwards compatible. This
  means that a program built against a C library using older kernel headers
  should run on a newer kernel (although it may not have access to new
  features), but a program built against newer kernel headers may not work
  on an older kernel.

  From: <https://www.kernel.org/doc/Documentation/kbuild/headers_install.txt>

[8.0] GCC
---------

[8.1] C compiler cannot create executables.
-------------------------------------------

This is almost always an error in your CFLAGS/CXXFLAGS. Ensure that you have
used -ONUM (CAPITAL O) and not (lowercase o) or (zero 0).

If the above doesn't fix the issue, try building the package with:

    $ CFLAGS= CXXFLAGS= LDFLAGS= kiss b pkg

You'll then be able to discern whether or not this was the issue.

[9.0] XORG
----------

[9.1] Why can't I start Xorg as a normal user?
----------------------------------------------

This is likely due to you running 'modprobe amdgpu' or equivalent post (or late)
boot which causes udev to miss setting up the GPU device in /dev/dri.

There are three separate solutions ranging from easy to hard. Either of the
three will solve the issue.

1.  Start the udevd service.

    This will keep udevd alive and ensure that it does its thing when it detects
    that the GPU driver has loaded.

    Enabling the service also turns on device hotplug and other features. This
    isn't the default as there's no need for the daemon to run on a static
    system.

        $ ln -s /etc/sv/udevd/ /var/service

2.  Run 'modprobe' earlier in the boot process.

    Running 'modprobe' in your inittab or in rc.d will result in it executing
    _after_ udevd has finished setting up devices.

    By moving 'modprobe' to /etc/rc.conf, it will load the driver before udevd
    executes which will allow it to detect and setup the device.

3.  Compile the GPU driver and firmware into the kernel.

    This is trickier but the best solution overall. Instead of using modules,
    compile the firmware into the kernel.

    You'll then not have to worry about udevd/modprobe as the kernel will
    automatically load the driver as early as possible.

    This may include compiling proprietary firmware into the kernel as well
    which is a little bit finicky but doable.


[9.2] How do I take a screenshot?
---------------------------------

Most users will typically use 'ffmpeg' as their screenshot tool as they'll
already have ffmpeg installed for MPV or Firefox. Other options include 'scrot',
  'imlib2' or 'imagemagick' (import).

If you'd like to use another tool, package it yourself. Remember, if no one has
packaged something it means that no one has needed said package. It is up to
you.
