SOFTWARE
========

Linux distributions are more or less a package manager, repositories of software
and some sugar on top. The software choices made by a distribution are largely
what define it.

KISS takes a different approach to its software offerings by having highly
focused defaults with limitless user choice as a counterbalance. The
defaults are just that, defaults. Nothing is tightly coupled.

This page will explain the many software choices, the status of packages like
Python 2 and the further removal of unneeded software. This is only in relation
to the official repositories as the Community make KISS boundless.

[1.0] The Defaults
------------------

KISS provides a somewhat non-typical software stack. It's small in size and
contains little software. As a user, you may find that some things work
differently to other distributions.

This is not a complete list, it is merely the interesting bits. The true
strength of KISS is not in its software choices but in its software omissions.

[1.1] C/POSIX Library
---------------------

Most major distributions tend to use GNU's C library whereas KISS uses musl.

Musl has better support for static linking, is smaller in size and has an
emphasis on correctness + conformance to standards. The dynamic runtime is a
single file with a stable ABI allowing for worry-free and race-free updates.

It is not without its caveats though. Software compiled for GNU's C library will
not work on a musl-based system. Large amounts of software also rely on library
extensions which are exclusive to the GNU C library.

Making this software work on musl is not impossible however. A tiny patch is
all that is needed in the majority of cases and this overall situation has
improved immensely in recent years.

To paint a clearer picture; our Firefox package no longer requires a series of
patches for musl compatibility as the portability issues were fixed upstream
(Thanks to $/michaelforney).

More information: https://wiki.musl-libc.org/design-concepts.html

[1.2] Core Utilities and Shell
------------------------------

The default provider of the core utilities (ls, cat, etc) and the POSIX shell is
busybox. Distributions typically use the GNU coreutils and bash to fill this
hole.

Busybox combines tiny versions of each utility into a single, small executable.
This is statically linked in KISS and comes in at 1MB for the entire package.
Each utility is simply a symlink to the single binary.

More information: https://www.busybox.net/about.html

[1.3] Init System and Service Manager
-------------------------------------

Busybox is also the provider of KISS' default init and service manager. This
works really well as no additional software is required and the overall system
integration is very minimal.

The service manager provided by busybox is its own implementation of the runit
family of utilities. Writing services for this system is really nice as services
are no larger than a line or two a file.

The boot-up and power-off procedures however, are not covered by busybox and
must be implemented externally. KISS provides an init-agnostic and portable base
with which any init and service manager can be used.

More information: $/kiss-community/init

[1.4] SSL Library
-----------------

KISS' default SSL library is LibreSSL, an OpenSSL fork by the OpenBSD folk. The
purpose of the fork is to modernize OpenSSL, improve security and apply
development best practices.

It also has a nicer build system and no reliance on Perl during compilation.
This made it very easy to include in KISS' tiny core.

More information: https://libressl.org/index.html

[2.0] Removed Software
----------------------

Through tireless effort, a large amount of software has been rendered unneeded
and the entire distribution works without their presence. Let's get the smaller
stuff out of the way first.

KISS has no need for:

    bash, dbus, fakeroot, file, ca-certificates, atk-brige-*, gettext,
    intltool, autoconf, automake, libtool, yasm, shared-mime-info, ...

Work is almost complete in removing more traditionally fundamental pieces of
software from the distribution. The following pieces of software have been
reduced to compile-time(!) dependencies of Firefox (and only Firefox).

*   Python 2 (Porting effort to Python 3 still ongoing upstream).
*   GTK+2    (Due to be removed upstream in 2020).
*   Perl     (Used throughout the build process. Tricky).

Trying to convey what software is excluded from the distribution is quite the
difficult task. It might make things clearer if I mention that the official
repositories are made up of only 150 or so packages (and this number has shrunk
over time).

[3.0] Excluded Software
-----------------------

You might be surprised to hear that a lot of popular software is explicitly
excluded from the official repositories. What this means is that they will never
make their way into the distribution officially.

The reasons why will be explained below, the (by no means complete) list:

*   dbus
*   elogind
*   polkit
*   pulseaudio
*   pam
*   wayland
*   All Desktop Environments.

KISS' official repositories exclude this software to ensure that the
distribution is fully functional without them. The word "fully" implies
everything up to and including a web browser (Firefox) and a media player (mpv).

It is a easier to add software to a system than it is to try and pry it out of
one. This also ensures that this software remains _optional_ and is not forced
onto users. Choices must remain so.

The Community have packaged this entire list (and then some) for those looking
to use this software. This is #1 the strength of the distribution: a highly
opinionated and minimal base with limitless extensibility.

See: <{{ site.wiki }}/community/repositories>
