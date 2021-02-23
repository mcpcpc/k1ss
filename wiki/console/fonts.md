CHANGING THE CONSOLE FONT
=========================

The Linux console font can be changed using the setfont utility (part of busybox
and installed by default).

Available Fonts
---------------

Currently packaged Linux console fonts.

*   terminus-font (Community).
*   spleen-font (Community).
*   unifont (Community).

Changing the Console Font
-------------------------

Use the following command to set the console font _after_ a user has logged in.
Add it to your ~/.profile or /etc/profile to make the change permanent.

    [ "$DISPLAY" ] || setfont /usr/share/consolefonts/FONT_NAME

To change the console font _before_ a user has logged in, add the following to
your /etc/inittab or /etc/rc.conf.

    # /etc/rc.conf
    /usr/bin/setfont /usr/share/consolefonts/FONT_NAME -C /dev/tty1

