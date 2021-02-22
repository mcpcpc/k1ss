EFIVARFS
========

To use efibootmgr and other software to manipulate the UEFI boot entries, the
efivars filesystem must be mounted. This isn't handled automatically by KISS due
to the security implications in doing so [0],[1].

    # Mount efivars.
    $ mount -t efivarfs none /sys/firmware/efi/efivars/

    # Unmount efivars.
    $ umount /sys/firmware/efi/efivars/

[0]: https://www.kernel.org/doc/Documentation/filesystems/efivarfs.txt
[1]: https://github.com/systemd/systemd/issues/2402> "Mount efivarfs read-only
