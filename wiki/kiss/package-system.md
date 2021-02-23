KISS PACKAGE SYSTEM
===================

The distribution employs a package system based around the concept of easily
parseable plain-text files (with fields separated by lines and spaces).

This format allows effortless interface using any programming language or just
basic UNIX utilities and tools.

[1.0] Directory Structure
-------------------------

    zlib/            # Package name.
    - build          # Build script (chmod +x).
    - sources        # Remote and local sources.
    - version        # Package version.

    # Optional files.
    - depends        # Dependencies (usually required).
    - pre-remove     # Pre-remove script (chmod +x).
    - post-install   # Post-install script (chmod +x).
    - patches/*      # Directory to store patches.
    - files/*        # Directory to store misc files.

[2.0] build
-----------

The build file should contain all of the steps necessary to patch, configure,
make and install ('make install' in this context) the package. The file is
language agnostic and the only requirement is that it be executable.

On execution the script will start in the directory of the package's source
(no need to cd anywhere).

The script is also given arguments (equivalent to 'script arg arg'). The first
argument contains the path where the script should install the compiled files.
The second argument contains the package's version.

Everything in the mentioned path is added to the package tarball and later
installed to the system.

Build:

    #!/bin/sh -e

    # Disable stripping (add this only if needed).
    :> nostrip

    ./configure \
         --prefix=/usr

    make
    make DESTDIR="$1" install

[3.0] depends
-------------

The depends file should contain any other packages the package depends on to
function correctly. Each dependency should be listed one per line.

A second and optional field allows for the distinction between a compile time
dependency and a run time one. The value of this field should be set to 'make'
for compile time dependencies and left blank for runtime dependencies.

The depends file can be omitted entirely if the package has no dependencies.

Depends:

    # Comments are supported.

    # Perl is needed at compile time.
    perl make

    # zlib is needed at runtime.
    zlib

[4.0] sources
-------------

The sources file should contain all remote and local files needed by the
package during the build. This includes the source, patches and any other
miscellaneous files which may be needed.

Each source should be listed one per line. An optional second field specifies
an extraction directory (relative to the build directory).

Sources which pull from a git repository use a special syntax. An additional
prefix 'git+' should be added with an optional suffix #hash or @branch).

Adding a suffix of '?no-extract' to the source URL will prevent the package
manager from automatically extracting the source (if a tarball or zip).

Sources:

    https://example.com/dhcpcd-8.0.2.tar.xz
    files/dhcpcd.run

Sources (usage with second field):

    https://example.com/gcc-9.1.0.tar.xz  gcc
    https://example.com/gmp-6.1.2.tar.xz  gcc/gmp
    https://example.com/mpfr-4.0.2.tar.xz gcc/mpfr
    https://example.com/mpc-1.1.0.tar.gz  gcc/mpc

Sources (usage with Git):

    git+https://github.com/dylanaraps/eiwd

    # Grabbing a specific branch.
    git+https://github.com/dylanaraps/eiwd@branch

    # Grabbing a specific commit.
    git+https://github.com/dylanaraps/eiwd#commit

Sources (usage with automatic source extraction disabled):

    https://example.com/zlib-1.2.0.tar.xz?no-extract

[5.0] version
-------------

The version file should contain a single line with two fields. All other lines
will be ignored.

The first field should contain the software's upstream version and the second
field should contain the version number of the repository files themselves.

If the package is using a git source to pull down the latest commit, the
version should be simply set to 'git'.

Version:

    1.2.3 1

Version (Git):

    git 1

[6.0] pre-remove
----------------

The pre-remove file should contain anything that needs run prior to the
removal of the package. The file is language agnostic and the only requirement
is that it be executable.

Pre-remove:

    #!/bin/sh -e

    printf 'No one has used this yet!\n'
    printf 'I have no example!\n'

[7.0] post-install
------------------

The post-install file should contain anything that needs to be run after a
package installation to properly setup the software. The file is language
agnostic and the only requirement is that it be executable.

Post-install:

    #!/bin/sh -e

    fc-cache -vf

[8.0] patches/*
---------------

The patches directory should contain any patches the software needs.

In the `sources` file patches are referred to by using a relative path
(patches/musl.patch). The package manager does not automatically apply patches.
This must be done in the build script of the package.

The build script has direct access to the patches in its current working
directory ('patch -p1 < example.patch' works directly).

Sources (usage with patches):

    https://example.com/rxvt-unicode-9.22.tar.bz2
    patches/gentables.patch
    patches/rxvt-unicode-kerning.patch

Build (usage with patches):

    #!/bin/sh -e

    patch -p1 < rxvt-unicode-kerning.patch
    patch -p1 < gentables.patch

    ./configure \
         --prefix=/usr

    make
    make DESTDIR="$1" install

[9.0] files/*
-------------

The files/ directory should contain any miscellaneous files the software needs.

In the sources file, files are referred to by using a relative path
(files/busybox.config).

The build script has direct access to the files in its current working
directory ('cat example_file' works directly).

Sources (usage with local files):

    https://example.com/kiss-1.0.0.tar.gz
    files/kiss_path.sh

Build (usage with local files):

    #!/bin/sh -e

    install -D kiss "$1/usr/bin/kiss"

    # Accessible directly in '$PWD'.
    install -D kiss_path.sh "$1/etc/kiss_path.sh"
