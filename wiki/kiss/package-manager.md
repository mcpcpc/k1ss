KISS PACKAGE MANAGER
====================

The KISS package manager is a self-contained POSIX shell script which is written
in a highly portable way. The entire utility comes in at under 1000 lines of
code (excluding blank lines/comments of which there are many).

The package manager is merely an implementation of the package format, its
requirements and some added sugar on top.

Source: $/kiss-community/kiss

[1.0] Usage
-----------

kiss:

    -> kiss [a|b|c|d|i|l|r|s|u|v] [pkg]...
    -> alternatives   List and swap to alternatives
    -> build          Build a package
    -> checksum       Generate checksums
    -> download       Pre-download all sources
    -> install        Install a package
    -> list           List installed packages
    -> remove         Remove a package
    -> search         Search for a package
    -> update         Update the system
    -> version        Package manager version

[2.0] Dependencies
------------------

POSIX utilities are used where appropriate _and_ where they exist to solve a
particular problem. Utilities of which there is only a single and cross-platform
implementation are considered "portable" (git, curl, etc)

If a dependency can be made optional, it will be made so. Dependencies are also
kept to a minimum (though we must also remain realistic).

| Dependency       | Reason for use                                 | Required |
|------------------|------------------------------------------------|----------|
| POSIX utilities  | Used throughout                                | Yes      |
| git              | Remote repositories and                        | Yes [1]  |
|                  | package git sources                            |          |
| curl             | Source downloads                               | Yes      |
| gpg1 or gpg2     | Repository signing                             | No       |
| sha256sum        | Checksums                                      | Yes [2]  |
| tar              | Sources, packages, etc                         | Yes [3]  |
| unzip            | Zip sources (very rare)                        | No       |
|                  |                                                |          |
|                  |                                                |          |
|------------------|------------------------------------------------|----------|
| Compression      |                                                |          |
|------------------|------------------------------------------------|----------|
| gzip, bzip2, xz  | Tarball compression                            | Yes [4]  |
| zstd, lzma, lzip | Tarball compression                            | No       |
|                  |                                                |          |
|                  |                                                |          |
|------------------|------------------------------------------------|----------|
| Privileges       |                                                |          |
|------------------|------------------------------------------------|----------|
| su | sudo | doas | Privilege escalation                           | No [5]   |
|                  |                                                |          |
|                  |                                                |          |
|------------------|------------------------------------------------|----------|
| Binutils         |                                                |          |
|------------------|------------------------------------------------|----------|
| ldd              | Dependency Fixer                               | No [6]   |
| strip            | Binary Stripping                               | No [6]   |
|                  |                                                |          |

[1] Git is also required for contribution to the distribution itself. Strictly
    speaking, nothing forces you to use git. Remote repositories and git based
    sources will simply become unusable.

[2] There is no standard utility for the generation of sha256 checksums. While
    sha256sum is listed above, the package manager also supports sha256, shasum
    and openssl as fallbacks.

[3] The tar command has no standard! This came as a shock. The POSIX equivalent
    is "pax" though this isn't in wide use (at least on Linux).

    Our usage of tar is merely, cf, xf and tf. A patch is applied to sbase's
    tar so that it supports "dashless" arguments (as all others do). Our usage
    of tar cannot become any more basic than it is now. Portability should no
    longer be a concern.

    Tested tar implementations include: busybox, toybox, sbase, GNU and
    libarchive (though all tar implementations should work in theory).

[4] These three compression methods are required as pretty much every package
    source uses them as the tarball compression method.

    The other compression methods are optional as no package sources (in the
    official repositories) make use of them.

    If a compression method has 1-3 uses (hasn't yet happened), the compression
    method will simply become a 'make' dependency of the package until usage
    increases to a "normality".

[5] A privilege escalation utility is only needed when using the package
    manager as a normal user for system-wide package installation.

    Installation to a user-writable directory does not require root access.

    Root usage of the package manager (chroot usage for example) does not
    require these utilities.

[6] If these are missing, binary stripping and/or the dependency fixer will
    simply be disabled.

    Regarding 'strip'; It has a POSIX specification, though the spec doesn't
    contain any arguments whatsoever.

    This makes our usage of 'strip' non-POSIX. That being said, our usage is
    compatible with these 'strip' implementations.

    strips: binutils, elfutils, elftoolchain, llvm, etc.

[3.1] Runtime dependency detector built around 'ldd'
----------------------------------------------------

Dynamic dependencies brought in by build systems (which are missing from the
package's dependency list) are fixed on-the-fly by checking which libraries
link to the package's files.

This prevents an incomplete dependency list from causing system breakage as
the package manager is able to complete the list.

A lot of packages make use of this "implicit" to "explicit" dependency list
"conversion" to provide optional dependencies.

Example output:

    -> libXmu Checking for missing dependencies
    --- /home/dylan/conf/cache/kiss/build-4477/d
     depends
    @@ -1,3 +1,8 @@
    +libX11
    +libXau
     libXext
     libXt
    +libxcb
     xorg-util-macros make

[3.2] Fully dynamic (and automatic) alternatives system
-------------------------------------------------------

Any file conflicts between two packages automatically become choices in the
alternatives system.

This allows one to swap providers of files without needing to explicitly
tell the package manager that two packages conflict, provide the same
utilities, etc.

In other words, no changes need to be made to packages. In fact, nothing
needs to be done at all. It's entirely automatic.

List available alternatives ('a' is an alias to 'alternatives'):

    $ kiss a
    gnugrep /usr/bin/grep
    ncurses /usr/bin/clear

Swap to GNU grep:

    $ kiss a gnugrep /usr/bin/grep
    -> Swapping '/usr/bin/grep' from busybox to gnugrep

Swap back to busybox grep:

    $ kiss a busybox /usr/bin/grep
    -> Swapping '/usr/bin/grep' from gnugrep to busybox

Swap to all alternatives for a given package (sbase for example).

    $ kiss a | grep ^sbase | kiss a -
    -> Swapping '/usr/bin/cat' from busybox to sbase
    -> Swapping '/usr/bin/cut' from busybox to sbase
    -> Swapping '/usr/bin/yes' from busybox to sbase
    ...Many more lines of output...

The above command works as the output of the alternatives listing is
directly usable as input to 'kiss a'.

[3.3] 3-way handshake for files in /etc/
----------------------------------------

Files in /etc/ are handled differently to those elsewhere on the system. A
reinstallation or update to a package will not always overwrite these files.

Instead, a 3-way handshake happens during installation to determine how the
new /etc/ file should be handled.

If the user has made modifications to the file and those modifications
differ to the to-be-installed package's file, the file is installed with the
suffix '.new'

If the user hasn't touched the file, it will be automatically overwritten by
the package manager as it will contain updated/new contents..

If the user has touched the file but the file has not changed between
package versions, it will simply be skipped over.

Example (with sha256 checksums truncated to fit):

    -> opendoas Doing 3-way handshake for etc/doas.conf
    Previous: 1656eee66c235cb717f9f8f35aa9c3587bb768d7fe  etc/doas.conf
    System:   4a51871b3190fa74726ea2b12ffafb96f40c172b68  etc/doas.conf
    New:      1656eee66c235cb717f9f8f35aa9c3587bb768d7fe  etc/doas.conf
    -> Skipping etc/doas.conf

[4.0] Configuration
-------------------

The package manager has no configuration files and no changes need to be made to
the system prior to its use. While there is no configuration file, this does not
mean that there is no possibility for configuration.

The package manager can be configured via the use of environment variables. I
believe this to be the best configuration method (where realistic). Environment
variables can be set system-wide, per-user, conditionally, for a single
invocation, etc, etc.

They require little to no extra code in the package manager to support them.

|        Variable | Description                                                |
|-----------------|------------------------------------------------------------|
|                 |                                                            |
|       KISS_PATH | List of repositories. This works exactly like '$PATH'      |
|                 | (a colon separated list of paths).                         |
|                 |                                                            |
|      KISS_FORCE | Force installation/removal of package by bypassing         |
|                 | dependency checks, etc. Set to '1' to enable.              |
|                 |                                                            |
|     KISS_CHOICE | Set to '0' to disable the alternatives system and error on |
|                 | any detected file conflicts.                               |
|                 |                                                            |
|       KISS_HOOK | Hook into the package manager. Set to the full path to the |
|                 | script. See @/wiki/kiss/package-manager-hooks              |
|                 |                                                            |
|       KISS_ROOT | Where installed packages will go. Can be used to install   |
|                 | packages to somewhere other than '/'.                      |
|                 |                                                            |
|      KISS_COLOR | Enable/Disable colors. Set to '0' to disable colors.       |
|                 |                                                            |
|     KISS_PROMPT | Skip all prompts. Set to '0' to say 'yes' to all prompts   |
|                 | from the package manager.                                  |
|                 |                                                            |
|   KISS_COMPRESS | Compression method to use for built package tarballs       |
|                 | (defaults to 'gz'). Valid: bz2, gz, lzma, lz, xz, zst      |
|                 |                                                            |
|         KISS_SU | Force usage of a different sudo tool.                      |
|                 | Valid: su, sudo, doas                                      |
|                 |                                                            |
|      KISS_STRIP | Enable/Disable package stripping globally.                 |
|                 | Set to '0' to disable.                                     |
|                 |                                                            |
|      KISS_DEBUG | Keep temporary directories around for debugging purposes.  |
|                 | Set to '1' to enable.                                      |
|                 |                                                            |
|    KISS_KEEPLOG | Keep build logs around for successful builds and not just  |
|                 | failing ones. Set to '1' to enable.                        |
|                 |                                                            |
|     KISS_TMPDIR | Temporary directory for builds. Can be set to a tmpfs so   |
|                 | builds happen in memory.                                   |
|                 |                                                            |
|        KISS_PID | Use a reproducible cache naming scheme instead of          |
|                 | build-$pid. If set to test, the result will be build-test. |
|                 |                                                            |

There are also a myriad of "3rd-party" environment variables which control GCC,
Make, CMake, etc. These aren't used by the package manager. They're used by the
tools called by the package's build script.

|        Variable | Description                                                |
|-----------------|------------------------------------------------------------|
|  XDG_CACHE_HOME | Cache directory location.                                  |
|                 |                                                            |
|              CC | C   compiler.                                              |
|             CXX | C++ compiler.                                              |
|              AR | Archive tool.                                              |
|              NM | Symbol  tool.                                              |
|          RANLIB | Index   tool.                                              |
|                 |                                                            |
|          CFLAGS | C   compiler flags.                                        |
|        CXXFLAGS | C++ compiler flags.                                        |
|         LDFLAGS | Linker flags.                                              |
|       MAKEFLAGS | Make flags.                                                |
|       SAMUFLAGS | Samurai flags.                                             |
|       RUSTFLAGS | Rust compiler flags.                                       |
|                 |                                                            |
| CMAKE_GENERATOR | 'Unix Makefiles' or 'Ninja'.                               |
|                 |                                                            |

[5.0] Repositories
------------------

Repository management in KISS is very simple. Repositories are configurable via
an environment variable. This environment variable can be set system-wide,
per-user, conditionally (via a script or program), for a single invocation, etc.

The environment variable is called '$KISS_PATH' and is functionally identical to
the '$PATH' variable. A colon separated list of paths in other words.

Example KISS_PATH:

    $ KISS_PATH=/var/db/kiss/repo/core:/var/db/kiss/repo/extra

In the above example, two repositories are enabled (Core and Extra). The package
manager will search this list for packages in the order it is written.

Repositories can live anywhere on the system. In your '$HOME' directory,
somewhere system-wide in '/', etc. The only requirement is that a full path be
used.

[5.1] What is a repository?
---------------------------

A KISS repository is simply a directory of directories. The quickest way to get
started is as follows.

Step 1 - create the repository:

    $ mkdir -p repo
    $ cd repo

Step 2 - let's 'fork' a few packages into our new repository:

    $ kiss fork curl
    $ kiss fork xz
    $ kiss fork zlib

This is now a fully usable repository and it can be added to your KISS_PATH.

[5.2] Enabling a remote repository
----------------------------------

Let's assume that our KISS_PATH matches the above example (Core and Extra). As
an example, we'll be enabling the Community repository.

Step 1 - clone the repository:

    # This can live anywhere on the system.
    $ git clone https://github.com/kiss-community/community

Step 2 - enable the repository:

    $ export KISS_PATH=$KISS_PATH:/path/to/community/community

[5.3] Preventing a package from receiving updates
-------------------------------------------------

Preventing a package from receiving updates can be accomplished in a myriad of
different ways. The easiest method is to leverage a user repository.

Step 1 - create a new repository:

    $ mkdir -p no_updates
    $ cd no_updates

Step 2 - copy the package to the new repository:

    $ cp -r /var/db/kiss/installed/PKG_NAME /path/to/no_updates

Step 3 - add the new repository to the /START/ of your KISS_PATH:

    $ export KISS_PATH=/path/to/no_updates:$KISS_PATH

The package manager will search KISS_PATH in order. It will see that the
no_updates repository provides PKG_NAME and the version matches that which is
installed. No updates will again happen for the package.

[5.4] Package fallbacks
-----------------------

If you would like to package something in your own repository but would like the
package manager to prefer other repositories before yours, simply add your
repository to the end of KISS_PATH.

The moment that your package is available elsewhere, the package manager will
prefer the new location to yours. The list is searched (in order) and the first
match is picked.

[5.5] Bypassing KISS_PATH
-------------------------

There is a special case where one can bypass the regular KISS_PATH for a
single invocation of the package manager. This has been called "CRUX-like
usage" by users.

    $ cd /path/to/myrepo/firefox
    $ kiss b
    $ kiss i

As seen above, various package manager commands will work without arguments,
so long as you are in a package's repository directory. This will prepend
the current directory to '$KISS_PATH' _only_ for this invocation.

[6.0] Package Manager Hooks
---------------------------

KISS' package manager is extensible via hooks which fire at various different
places inside the utility. Hooks allow the user to modify behavior, add new
features or conditionally do things on a per-package basis.

This setting is controlled by the '$KISS_HOOK' environment variable which takes
the full path to a POSIX shell script as its value.

Example:

    export KISS_HOOK=$HOME/.local/bin/kiss-hook

[6.1] Usage
-----------

The script is sourced at each hook and is given three variables as input.
POSIX shell does not allow sourced scripts to receive arguments, these
variables are instead defined via the script's environment.

- '$PKG':  The name of the current package.
- '$TYPE': The type of hook.
- '$DEST': The location where 'make install' will put files.

Valid values for '$TYPE' include:

    pre-build,   post-build,   build-fail,
    pre-install, post-install, pre-extract,
    post-package.

As the hook script is sourced (instead of being executed in its own shell),
the script has the ability to override package manager internals and the
package manager's environment.

[6.2] Removing unneeded files from packages
-------------------------------------------

Packages can contain files which you will have no use for. A simple hook can
be defined to remove them from packages.

> NOTE: This is the default 'KISS_HOOK' script. If defining your own, be sure
>       to include this if you would like to continue to remove these files.

    case $TYPE in
        post-build)
            # Ensure that '$DEST' is set.
            : "${DEST:?DEST is unset}"

            rm -rf "$DEST/usr/share/gettext" \
                   "$DEST/usr/share/polkit-1" \
                   "$DEST/usr/share/locale" \
                   "$DEST/usr/share/info" \
                   "$DEST/usr/lib/charset.alias"
        ;;
    esac

[6.3] Building some packages in memory (tmpfs)
----------------------------------------------

This hook runs on pre-extract and conditionally runs the source extraction,
package build process and resulting tarball creation in memory. The benefit of
this is a nice speedup throughout (especially when used alongside ccache [0]).

> NOTE: '/tmp' must be mounted as 'tmpfs' for this to work.

    case $TYPE in
        pre-extract)
            case $PKG in
                # Run everything on disk for these memory hungry packages.
                firefox|firefox-esr|rust|llvm|clang)
                    mak_dir=$KISS_TMPDIR/build-$pid
                    pkg_dir=$KISS_TMPDIR/pkg-$pid
                ;;|

                *)
                    log "$PKG" "Activating tmpfs"|

                    mak_dir=/tmp/build-$pid
                    pkg_dir=/tmp/pkg-$pid|
                ;;
            esac

            mkdir -p "$mak_dir" "$pkg_dir/$PKG/var/db/kiss/installed"
        ;;
    esac

[6.4] Logging build duration
----------------------------

This hook adds a new message post-build with the total build duration in a human
readable format (00h 00m). Similar code is used in the boot process of the
system to calculate boot time.

    case $TYPE in
        pre-build)
            IFS=. read -r _start _ < /proc/uptime
        ;;

        post-build)
            IFS=. read -r _end _ < /proc/uptime

            (
                _s=$((_end - _start))
                _h=$((_s / 60 / 60 % 24))
                _m=$((_s / 60 % 60))

                [ "$_h" = 0 ] || _u="${_u}${_h}h "
                [ "$_m" = 0 ] || _u="${_u}${_m}m "

                log "$PKG" "Build finished in ${_u:-${_s}s}"
            )
        ;;
    esac

[7.0] Package Manager Extensions
--------------------------------

Anything in the user's '$PATH' which matches the glob 'kiss-*' will be directly
usable via the package manager. For example, 'kiss-size' is also usable as
'kiss size' (and even 'kiss si') (the shortest available alias).

The detected 'kiss-*' utilities will appear in the package manager's help output
with the second line in the script acting as a doc-string.

Example help output (kiss extensions):

    -> Installed extensions (kiss-* in $PATH)
    -> chbuild    Create/destroy temporary chroots
    -> chroot     Enter a kiss chroot
    -> depends    Display a package's dependencies
    -> export     Installed package to tarball
    -> fork       Fork a package into the current dir
    -> help       Read KISS documentation
    -> link       Link a repo file to another repo
    -> maintainer Find the maintainer of a package
    -> manifest   Display all files owned by a package
    -> new        Create a boilerplate package
    -> orphans    List orphaned packages
    -> outdated   Check repository packages for updates
    -> owns       Check which package owns a file
    -> reset      Remove all packages but base
    -> revdepends Packages which depend on package
    -> size       Show the size on disk for a package

These are in effect, optional utilities which interact with the package system
in one way or another. My hope behind them is to act as an example as to how
easy it is to interface with the plain-text and "static" package system.

Example utility, kiss-depends (kiss depends, kiss de):

    #!/bin/sh -ef
    # Display a package's dependencies

    # Ignore shellcheck as we want the warning's behavior.
    # shellcheck disable=2015
    [ "$1" ] && kiss l "${1:-null}" >/dev/null || {
        printf 'usage: kiss-depends [pkg]\n'
        exit 1
    }

    cat "$KISS_ROOT/var/db/kiss/installed/$1/depends" 2>/dev/null

[8.0] Tips and Tricks
---------------------

A lot of the package manager's features are hard to discover or otherwise
non-obvious to its users. This section will document these features, how to use
them and the benefits they bring.

[8.1] Swap grep implementations for a major speed up.
-----------------------------------------------------

The default grep implementation in KISS is busybox grep. This version of grep
works very well and supports a large number of features. The one issue is that
it is painfully slow when compared to other popular implementations.

A fairly major speedup can be attained by swapping to a different grep via the
alternatives system. The fastest grep implementation around is GNU grep which is
available in Community as 'gnugrep'.

Step 1 - install GNU grep:

    $ kiss b gnugrep && kiss i gnugrep

Step 2 - swap to GNU grep:

    $ kiss a gnugrep /usr/bin/grep
