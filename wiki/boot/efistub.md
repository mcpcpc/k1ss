EFISTUB
=======

The Linux kernel has the ability to act as an EFI executable which simplifies
the boot process by removing the need for a bootloader [0].

Prerequisites
-------------

Begin by first verifying that you have efibootmgr installed:

    $ kiss b efibootmgr && kiss i efibootmgr

The kernel must be configured to support acting as an EFISTUB.

    CONFIG_EFI_STUB=y

The UEFI variables must be mounted.

See: <{{ site.wiki }}/efivarfs>

TIP: Keep your kernel's name as 'vmlinuz' to avoid having to update the
     EFI entries with each kernel update.

Create an UEFI boot entry
-------------------------

Use the following command to create a boot entry. Replace /dev/sdXN with your
EFI system partition and /dev/sdYM with your root partition. Check to see that
the entry was added with 'efibootmgr --verbose'.

    $ efibootmgr \
          --create \
          --disk     /dev/sdX \
          --part     N \
          --label    KISS \
          --loader   /vmlinuz \
          --unicode  'root=/dev/sdYM' \
          --verbose

The root partition can also be referred to by its partition UUID which can be
found using blkid.

          --unicode 'root=PARTUUID=aaaaaaaa-aaaa-4aaa-aaa-aaaaaaaaaaaa'

The above command, the --unicode option specifies the parameters passed to the
kernel. If you require other parameters to boot your system, pass them here.

Removing a UEFI boot entry
--------------------------

To remove an entry, find your entry's boot number using 'efibootmgr --verbose'
and run the following command.

    $ efibootmgr --bootnum XXXX --delete-bootnum

References
----------

[0]: https://www.kernel.org/doc/Documentation/efi-stub.txt
