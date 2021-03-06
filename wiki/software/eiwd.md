EIWD
====

eiwd is iNet Wireless Daemon (iwd) without dbus. iNet Wireless Daemon aims to
replace @/wpa_supplicant while providing the following benefits [0]:

*   Simplification of network management.
*   Faster network discovery.
*   Fast and reliable roaming.
*   Use less system resources.
*   Use features offered by the linux kernel.
*   Support for enterprise security methods like EAP.
*   Support for kernel asymetric key rings and trusted platform modules (TPM).
*   Support for multiple clients.

Configuration
-------------

Ensure that you have the following dependencies installed:

    $ kiss b eiwd && kiss i eiwd
    $ kiss b openresolv && kiss i openresolv

Create a new daemon configuration file:

    $ mkdir -p /etc/iwd
    $ touch    /etc/iwd/main.conf

Using a preferred text editor, add the following lines to the main.conf file
generated above:

    [General]
    EnableNetworkConfiguration=true

    [Network]
    RoutePriorityOffset=200
    NameResolvingService=resolvconf

Adding a wireless network
-------------------------

eiwd ships with a iwd_passphrase, which can be used for generating iwd network
files:

    $ printf PASSWORD | iwd_passphrase BSSID

Remember to replace BSSID and PASSWORD with your respective network credentials.

Using a preferred text editor, copy the output of the command above to the
following location: /var/lib/iwd/BSSID.psk

Managed via runsv
-----------------

Busybox's runsv can be used to create a new managed service with the following
command:

    $ ln -s /etc/sv/eiwd/ /var/service

To start the new managed service, use the following command:

    $ sv up eiwd

Tips and tricks
---------------

*   To prevent iwd from continuous scanning while not connected, add the
    following to your main.conf file:

        [Scan]
        DisablePeriodicScan=true

*   To prevent iwd from destroying / recreating wireless interfaces at startup,
    add the following to your main.conf file:

        [General]
        UseDefaultInterface=true

*   If iwd fails to start, check to make that you have the required kernel
    options:

        CONFIG_CRYPTO_USER_API_HASH
        CONFIG_CRYPTO_USER_API_SKCIPHER
        CONFIG_KEY_DH_OPERATIONS
        CONFIG_CRYPTO_ECB
        CONFIG_CRYPTO_MD5
        CONFIG_CRYPTO_CBC
        CONFIG_CRYPTO_SHA256
        CONFIG_CRYPTO_AES
        CONFIG_CRYPTO_DES
        CONFIG_CRYPTO_CMAC
        CONFIG_CRYPTO_HMAC
        CONFIG_CRYPTO_SHA512
        CONFIG_CRYPTO_ARC4
        CONFIG_CRYPTO_SHA1

[0]: https://github.com/dylanaraps/eiwd
[1]: https://wiki.gentoo.org/wiki/Iwd
[2]: https://manpages.debian.org/testing/iwd/iwd.config.5.en.html
