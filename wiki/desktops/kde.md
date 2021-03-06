KDE
===

KDE [0] is desktop environment which consists of the Plasma Desktop, the KDE
Frameworks, and a collection of free and open source, cross-platform programs.
The project makes integral use of the Qt framework [1] to help provide a
uniform experience for users between all of the software a user will install.

$/dilyn-corner/KISS-kde is still considered to be under heavy development. While
the desktop environment works quite well, many things have yet to be tested. If
you run into bugs, problems, or have suggestions, submit issues or pull requests
on the GitHub page!

NOTE: due to the KISS Guidestones [2], the $/dilyn-corner/KISS-kde project is
entirely community maintained. It will always remain separate from the core KISS
project, including the community repository. Specifically, KDE violates the
software inclusion guidestone. The required violations are dbus and wayland.
However, logind, polkit, and PAM are optional to build and use KDE.

[0.0] Getting Started
---------------------

To begin using the K Desktop Environment there are two choices. The first option
is to use a prebuilt tarball similar to the KISS release tarballs and install it
analogously to how one installs KISS. The second is to build every package from
source from an existing KISS installation. Beyond this, one need only clone two
git repositories, install dbus, and choose whether to use libudev-zero or eudev.

There are several optional features that users have a choice on. These choices
include authentication backends, login managers, and window managers. These
choices can be changed at any time, and will simply require rebuilding certain
packages. The following subsections offer an explanation of what these choices
are and how best to handle their consequences.

The first thing to do is clone the repositories and add them to your $KISS_PATH.
Assuming that you keep these repositories in $HOME,

    $ git clone https://github.com/kisslinux/community
    $ git clone https://github.com/dilyn-corner/KISS-kde

    $ export KISS_PATH="$HOME/KISS-kde/extra:$KISS_PATH"
    $ export KISS_PATH="$HOME/KISS-kde/plasma:$KISS_PATH"
    $ export KISS_PATH="$HOME/KISS-kde/frameworks:$KISS_PATH"
    $ export KISS_PATH="$KISS_PATH:$HOME/community/community"

    # you can optionally include the kde path for sddm, dolphin, etc.
    $ export KISS_PATH="$KISS_PATH:$HOME/KISS-kde/kde"

[0.1] Authentication
--------------------

KDE utilizes polkit and PAM for authentication. They provide a robust backend
and sets of features which allow for more fine-grained control of\which users
can perform what actions. However, there are no strict requirements to use them.
For instance, kauth (the framework responsible for providing access to the
polkit backend) only optionally requires polkit. Additionally, kscreenlocker
(the program responsible for locking the screen) only requires PAM in instances
where the login manager has failed to properly track user sessions and will not
allow the user to login.

In the case of single-seat or single-user machines, complicated authentication
frameworks might be seen as a needless additional form of security. If users
decide they do not wish to make use of these features, they are not required to
do anything.

In cases where users would prefer to have fine-grained controls over access, the
only prerequisite is to install a user permissions backend and polkit prior to
building the rest of KDE. The permission systems allowed are linux-pam or
shadow. If users intend on installing a login manager (such aselogind), PAM
should be chosen. Otherwise, shadow is an excellent option(and is available in
community).

If a user wishes to go this route, simply build and install linux-pam or shadow,
and then install polkit:

    $ kiss b linux-pam   && kiss i linux-pam
    $ kiss b polkit      && kiss i polkit
    $ kiss b polkit-qt-1 && kiss i polkit-qt-1

If users change their mind after setting up KDE, reverting is as simple as
rebuilding the relevant packages. For instance, if a user decided to build
polkit and linux-pam and wishes to drop the stack entirely,

    # Uninstall linux-pam and polkit
    $ KISS_FORCE=1 kiss r linux-pam polkit polkit-qt-1

    # Rebuild kauth, kscreenlocker
    $ kiss b kauth kscreenlocker
    $ kiss i kauth kscreenlocker

Known issues with lacking proper authentication backends includes the inability
to use systemsettings to alter things like the time.

NOTE: currently there seems to be an issue with authentication in general;
kscreenlocker does not allow users to log back in, and users cannot change the
date or time. Without linux-pam installed, the former issue is resolved.

[0.2] Session Management
------------------------

Currently, users can either start plasma sessions via TTY or from a greeter.
Login managers provide a way by which certain other programs can track sessions,
and provide a measure of security for the system. For instance, networkmanager
optionally requires logind for ensuring internet features return after the
system wakes up from sleep, and sddm allows restrictions on which UIDs can login
to a system.

For users who have limited interest in managing their sessions via logind, no
action is required.

For users who wish to have session-tracking features or would like to utilize
programs like sddm, elogind is required (other options are currently being
explored). elogind requires PAM. As a result, users choosing to use elogind
should do the following:

    # Uninstall shadow if it is installed
    $ KISS_FORCE=1 kiss r shadow

    # Install linux-pam
    $ kiss b linux-pam && kiss i linux-pam

    # Rebuild polkit
    $ kiss b polkit polkit-qt-1 && kiss i polkit polkit-qt-1

    # Install elogind
    $ kiss b elogind && kiss i elogind

    # Because of the way we build polkit, rebuild it
    $ kiss b polkit && kiss i polkit

See [4.2] on configuration options for sddm.

NOTE: to take advantage of the authentication backend, users should reinstall
kauth and kscreenlocker.

[0.3] Default Window Manager
----------------------------

Currently there are two primary window managers that can be chosen: kwin and
kwinft. kwinft [3] is a composited window manager for X11 and Wayland systems.
Most of the differences between kwinft and kwin are in the underlying code;
kwinft is designed to have better coding practices and be well-organized.
However, any features that appear in kwinft can be expected to eventually show
up in kwin. So if you prefer a more bleeding edge, potentially better window
manager for KDE, kwinft is an option.

By default, kwin will be installed. If you wish to change this, fork
plasma-{workspace,desktop} and alter the depends files. All you are required to
do is uncomment the kwinft lines in the dependencies lists and comment out kwin.
For more information on forking packages, see [4]. In order to change which is
used on an existing installation,

    $ pkill X

    $ KISS_FORCE=1 kiss r kwin kwayland-server

    $ kiss b kwinft && kiss i kwinft

    $ kiss b plasma-workspace && kiss i plasma-workspace
    $ kiss b plasma-desktop   && kiss i plasma-desktop

    $ startx

[1.0] From Scratch
------------------

Building KDE is a very straightforward process. In fact, if users already have a
working xorg setup, then it is as simple as building a single package - the
package manager will take care of the rest.

The package to install when going this route is plasma-desktop. This package
will pull in the required packages to have a fully working desktop environment,
and will additionally include a system settings manager, a default theme
(breeze), and some default icons (breeze-icons). There are some extra packages
which can be installed for more features, such as bluedevil, drkonqi, and
powerdevil. See [4.0] for more information.

[1.1] Prerequisites
-------------------

Ensure that you have the proper required packages installed: libudev-zero, dbus,
and xorg-server. libudev-zero has been tested to work with KDE. Optionally,
eudev also works. The choice is up to you! Currently, dbus-alternatives (like a
stub library) are untested. The dependencies of the KDE packages assume many
Xorg libraries are already installed. Not having xorg-server might result in
missing dependencies.

These are the only requirements for building plasma-desktop.

NOTE: in order to build certain extras, coreutils is required. Specifically:
elogind, libblockdev, and udisks2 require three separate programs. Patches are
welcome to resolve this! coreutils provides files which conflict with other
KISS packages, such as busybox. To ensure we are using the correct programs
when we attempt to build these extras, we use the alternatives system! For more
information on this system, see [5]. This is not required if you do not care
about elogind.

| Package     | Requirement                                                    |
|-------------|----------------------------------------------------------------|
|             |                                                                |
| elogind     | /usr/bin/realpath --relative-to                                |
| udisks2     | /usr/bin/ln -r                                                 |
| libblockdev | /usr/bin/mktemp --tmpdir                                       |
|             |                                                                |

NOTE: additionally, if desired, you must enable cgroup support in your kernel
for elogind. see [6] for more details.

Ensure the relevant prerequisites are installed:

    $ kiss b libudev-zero dbus
    $ kiss i libudev-zero dbus
    $ kiss b xorg-server libinput xf86-input-libinput
    $ kiss i xorg-server libinput xf86-input-libinput

    # If you opt to install e.g. elogind
    $ kiss b coreutils && kiss i coreutils
    $ kiss a coreutils /usr/bin/realpath
    $ kiss a coreutils /usr/bin/mktemp
    $ kiss a coreutils /usr/bin/ln

NOTE: If you had packages like xorg-server installed, you might want to rebuild
them to pickup on the new device manager they can take advantage of. If you do
not currently have xorg-server installed, you will want to get it. A pure
wayland KDE is both untested and unlikely to work.

NOTE: with the release of plasma 5.20.0, KDE is more aggressively supporting a
wayland environment. xorg support may be dropped in the future!

Make sure you do not currently have qt5 installed. The build files have been
tweaked for this repository to make use of dbus and libudev-zero. As a result,
carrying over the community version of these packages will lead to unknown
problems.

Because of the time required to build qt5* packages, some prebuilt tarballs are
available on the Github repository [5]. They are built in a standard KISS
tarball, with C(XX)FLAGS=-march=x86-64 -mtune=generic -Os -pipe.

NOTE: In general, it is a security risk to install prebuilt packages. these
archives are provided as a courtesy by $/dilyn-corner. Always verify the
authenticity of packages, and the identities of packagers!

The current release is 5.15.1, available with the 2020.10-1 release.

    $ ver=2020.10-1
    $ qtver=5.15.1-1
    $ url=https://github.com/dilyn-corner/KISS-kde/releases/download/$ver

    $ wget $url/qt5.$qtver.tar.gz \
           -O qt5@$qtver.tar.gz
    $ wget $url/qt5-declarative.$qtver.tar.gz \
           -O qt5-declarative@$qtver.tar.gz

    # Not a build-time requirement, but useful for e.g. falkon
    $ wget $url/qt5-webengine.$qtver.tar.gz \
           -O qt5-webengine@$qtver.tar.gz

It is recommended to verify the checksums for these tarballs to ensure package
integrity!

    $ wget $url/qt5.$qtver.tar.gz.sha256

    $ sha256sum -c < qt5@$qtver.tar.gz
    $ sha256sum -c < qt5-declarative@$qtver.tar.gz
    $ sha256sum -c < qt5-webengine@$qtver.tar.gz

    $ kiss i qt5@$qtver.tar.gz
    $ kiss i qt5-declarative@$qtver.tar.gz
    $ kiss i qt5-webengine@$qtver.tar.gz

[1.2] Installing
----------------

Now that all of the build requirements are taken care of, we can install the
desktop environment. Just over a hundred packages are required.

    $ kiss b plasma-desktop && kiss i plasma-desktop

Optionally, enable and start the dbus service:

    $ ln -s /etc/sv/dbus /var/service
    $ sv up dbus

No services are required to be running in order to start a plasma session.

[1.3] Launching
---------------

Once plasma-desktop has been installed, launching KDE is as simple as adding the
following to whatever script you use to start X:

    $ exec dbus-launch --exit-with-session startplasma-x11

The default fonts KDE usually ships with are hack and noto-sans. The former is
available in community, and the latter is included in KISS-kde/extra. Ensure
that you have a font installed prior to launching Plasma!

[2.0] The KISS-kde tarball
--------------------------

Similarly to KISS itself, a tarball containing a fully-functional KISS with KDE
setup is available from the GitHub repository [7]. This archive can bechrooted
into or directly unpacked to '/'. It is built from a kiss-chroot archive with
identical generic C(XX)FLAGS, and is ~330MB in size.

NOTE: In general, it is a security risk to install prebuilt packages. This
archive is provided as a courtesy by $/dilyn-corner. Always verify the
authenticity of packages, and the identities of packagers!

The KISS-kde archive is a fully installed version of plasma-desktop, but does
not make choices for the user. As a result, it should be installed identically
to how KISS is normally installed, using the KISS install guide. See [8] for
KISS setup instructions.

There will be monthly releases of the KISS-kde tarball to keep up with upstream.
The format of releases will be in YYYY-MM format.

To download e.g. the first October 2020 release,

    $ ver=2020.10-1
    $ url=https://github.com/dilyn-corner/KISS-kde/releases/download/$ver
    $ wget $url/kiss-kde-$ver.tar.xz

It is strongly recommended to verify the checksums to avoid problems like using
a partially downloaded archive.

    $ wget $url/kiss-kde-$ver.tar.xz.sha256
    $ sha256sum -c < kiss-kde-$ver.tar.xz.sha256

After setting up your disks, mount your desired root partition to '/mnt' and to
install the latest release,

    $ tar xf kiss-kde-$ver.tar.xz -C /mnt

and enter the chroot environment,

    $ /mnt/bin/kiss-chroot /mnt

From here, simply follow the rest of the install guide [8]. For help with
kernels, see <{{ site.wiki }}/kernel>. For help with bootloaders, see
<{{ site.wiki }}>/boot.

It is recommended that you change the root password; the default is "toor".

After setting up everything, it should be as simple as:

    $ exit
    $ reboot
    $ startx

You should now be greeted with a fresh Plasma Desktop!

[3.0] Starting KDE
------------------

You have two choices on how to launch KDE. Either you can start it directly from
the console, or you can make use of a login manager.

[3.1] Console
-------------

Launching a KDE session directly from the console is very straightforward. If
you would like to use wayland instead of X, replace x11 by wayland in the below
command.

Simply add a line to your xinitrc file:

    $ launch="exec dbus-launch --exit-with-session startplasma-x11"
    $ echo "$launch" >> "$HOME/.xinitrc"

It may be useful to set a runtime directory. A wayland session requires this be
set, and an X session will default to /tmp if you do not set one. Add this
example to somewhere like .profile:

    export XDG_RUNTIME_DIR="${XDG_RUNTIME_DIR:-/tmp/$(id -u)-runtime}"

    [ -d "$XDG_RUNTIME_DIR" ] || {
        mkdir -p   "$XDG_RUNTIME_DIR"
        chmod 0700 "$XDG_RUNTIME_DIR"
    }

[3.2] Login Manager
-------------------

The KISS-kde repository also includes sddm [9], the Simple Desktop Display
Manager, as an option for a greeter. It requires a login manager. elogind is the
default; other options are currently being tested.

First, build and install elogind:

    $ kiss b elogind && kiss i elogind
    # So that polkit will build the correct libs to support elogind,
    $ kiss b polkit && kiss i polkit

Then, install sddm:

    # sddm is in the KISS-kde/kde repo
    $ kiss b sddm && kiss i sddm

NOTE: by default, it may not be possible for root to login via sddm. Best
practices would dictate that you login as an unprivileged user.

Enable the requisite services for sddm to function properly. For KISS' default
service manager,

    $ ln -s /etc/sv/polkitd /var/service
    $ ln -s /etc/sv/elogind /var/service
    $ ln -s /etc/sv/sddm    /var/service

    $ sv up polkitd
    $ sv up elogind
    $ sv up sddm

If everything went well, sddm should immediately launch after you start the sddm
service

[4.0] Post-Install
------------------

Everything you do from here on is to customize your new Plasma Desktop to your
own needs! No further setup or configuration is required. However, there are a
few things available to you that you can change, if you so choose.

[4.1] Window Managers
---------------------

It is possible to use your favorite window manager with KDE, and it is
shockingly simple. All that is required is to install whatever window manager
you would prefer (dwm, i3, sowm, etc.), and add the following to your
environment:

    # replace 'sowm' with whichever wm you prefer
    $ export KDEWM=sowm

A 'good place' is up to the user; for per-user choice, add it to .profile or
your xinitrc file. To use the choice system-wide, add it to /etc/profile or
/etc/X11/xinit/xinitrc.

[4.2] Greeters
--------------

sddm is a feature-rich but straightforward greeter; see [3.2] for  installation
and setup instructions. sddm can be configured heavily. Configuration is read
from /etc/sddm.conf. If it does not exist, you can generate it:

    $ sddm --example-config >> /etc/sddm.conf

You can manually edit this file to allow for autologins, changing themes,
changing the user icons, or even changing the range of UIDs that can login
through sddm.

Alternatively, configuration can be done graphically during a plasma session.
Simply install sddm-kcm and a sddm configuration section in systemsettings
should become available.

    $ kiss b sddm-kcm && kiss i sddm-kcm

NOTE: other greeters are untested. If you find success with alternatives, make a
PR to add them!

[4.3] Extras
------------

Several things are included in KISS-kde/kde. Examples include sddm, kvantum [10]
and latte-dock [11].

Separate from these are the KDE Applications, such as krita or dolphin. Some
useful (and working) applications include dolphin and konsole.

Finally, there are several useful extra programs which will enhance the standard
KDE experience.

| Package     | Description                                                    |
|-------------|----------------------------------------------------------------|
|             |                                                                |
| sddm        | The Simple Desktop Display Manager                             |
| kvantum     | An svg-based theme engine for qt5                              |
| latte-dock  | A feature-rich dock based on the Plasma Framework              |
|             |                                                                |
| dolphin     | The default KDE file manager                                   |
| konsole     | The default KDE terminal emulator                              |
|             |                                                                |
| baloo       | A framework for file indexing and metadata management          |
| drkonqi     | A useful crash handler                                         |
| udisks2     | A disk manager                                                 |
| kgamma5     | Change the monitor's gamma                                     |
| khotkeys    | Expanded hotkey modification                                   |
| bludevil    | Bluetooth integration                                          |
| powerdevil  | Power usage settings                                           |
| kinfocenter | Displays useful system information                             |
|             |                                                                |

[5.0] Troubleshooting
---------------------

Here is a collection of some common issues you might encounter in buiding or
running KDE. If you encounter any other problems, please submit an issue at
$dilyn-corner/KISS-kde.

[5.1] My fonts don't work!
--------------------------

If the fonts you have installed don't appear in any Qt applications, such as
konsole, but DO appear in other applications, such as st, Qt is not properly
identifying your font location. This can be confirmed by running a Qt app from a
terminal and seeing warnings about how Qt does not ship fonts, and it could not
find them in /usr/lib/fonts. Additionally, qtdiag may inform you that the
default system font is noto-sans, regardless of whether or not noto-sans is
installed.

This issue should be resolved by merely rebuilding qt5.

[5.2] I can't login from a screen lock!
---------------------------------------

This is a known issue with kscreenlocker and KISS. The problem would seemingly
lie with PAM not properly authenticating the user. This remains unresolved.

If the user trying to login is root, you may have luck killing the
kscreenlocker_greet process.

The short term fix is to disable screen locking during inactivity, not require a
password while unlocking, or to simply never lock the screen.

This issue does not exist on systems which do not have linux-pam installed.

[5.3] I can't change the system time!
-------------------------------------

Administrative actions currently cannot be executed as any user in places such
as systemsettings.

A solution to resolve this issue is currently unknown.

[5.4] XYZ package fails to build!
---------------------------------

Please open an issue on the GitHub page! The dependencies have been minimized as
much as possible to ensure the fewest things required are pulled in to have a
working system. As a result, some things may fail tobuild independently. Some
dependency requirements are simple fixes. Some examples include:

| required pkg  | Error                                                         |
|---------------+---------------------------------------------------------------|
|               |                                                               |
| pkgconf       | ...pkg foo not found...                                       |
| linux-headers | linux/bar.h not found                                         |
|               |                                                               |

[6.0] How You Can Help
----------------------

The KISS-kde project is always looking for more contributors. Whether it is
fixing build errors, submitting patches, or adding applications, every
contribution is welcome!

For a current list of project milestones, check out the first section of the
README at $/dilyn-corner/KISS-kde.

[0]: https://kde.org
[1]: https://qt.io
[2]: https://kiss.armaanb.net/guidestones
[3]: https://gitlab.com/kwinft/kwinft
[4]: https://kiss.armaanb.net/package-manager#3.2
[5]: https://kiss.armaanb.net/package-manager#5.3
[6]: http://linuxfromscratch.org/blfs/view/svn/general/elogind.html
[7]: https://github.com/dilyn-corner/KISS-kde/releases
[8]: https://kiss.armaanb.net/install
[9]: https://wiki.archlinux.org/index.php/SDDM
[10]: https://github.com/tsujan/Kvantum
[11]: https://github.com/KDE/latte-dock
