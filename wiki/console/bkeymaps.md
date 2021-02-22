BKEYMAPS 
========

Binary keymaps to set the keyboard layout [0].

Install keymaps
---------------

Begin by verifying that you have bkeymaps installed from the Community
repository.

    $ kiss b bkeymaps && kiss i bkeymaps

Available keymaps are in /usr/share/bkeymaps.

Load Keymaps at login
---------------------

To load a keymap at login, add the following command to your .profile.

    # Load keymap using busybox's loadkmap.
    $ loadkmap < file                                                          

    # Load keymap using kbd's loadkeys.                                            
    $ loadkeys file

References
----------
    
[0]: https://dev.alpinelinux.org/bkeymaps/
