REPLACING UDEV
==============

As of the 20/01/2020, it is now possible to replace eudev with the device
manager of your choosing. This Wiki page will cover replacing eudev with
busybox mdev, however the steps are more or less the same for all other
device managers (smdev, vdev, ...).

Caveats
-------

*   Xorg will be unable to automatically detect input devices.
*   Libinput will be unable to use its quirks system.

Benefits
--------

*   Use any device manager, swap between them or use none at all.
*   Alternatives are simpler and lighter.
*   Faster boot process.

Preparation
-----------

1.  Make note of your current XKB rules, model and layout.

    Xorg compiled without eudev may require manual configuration to correctly
    set the keyboard's layout and other settings. The following command can be
    used to detect the current configuration from a working machine.

        $ setxkbmap -query
          rules:      evdev
          model:      pc105
          layout:     us

2.  Make note of your keyboard and mouse's /dev/input/event* entries.

    We also need to point Xorg to the input device's device entries in
    /dev/input. The following command lists all current input devices.

        $ xinput
        | Virtual core pointer                id=2    [master pointer  (3)]
        |   | Virtual core XTEST pointer      id=4    [slave  pointer  (2)]
        |   | touchpad0                       id=6    [slave  pointer  (2)]
        | Virtual core keyboard               id=3    [master keyboard (2)]
            | Virtual core XTEST keyboard     id=5    [slave  keyboard (3)]
            | keyboard0                       id=7    [slave  keyboard (3)]

    My keyboard has the ID '7'. The following command prints its information
    including the needed device node location.

        $ xinput --list-props 7
        Device 'keyboard0':
        Device Enabled (169):   1
        libinput Send Events Modes Available (316):     1, 0
        libinput Send Events Mode Enabled (317):        0, 0
        libinput Send Events Mode Enabled Default (318):        0, 0
        Device Node (319):      "/dev/input/event4"
        Device Product ID (320):        1, 1

    The "Device Node (XXX):" line displays the location in /dev/ of my keyboard
    and is the value we will use when configuring Xorg. My keyboard is located
    at /dev/input/event4. This command should be repeated for any other input
    devices.

Configuring Xorg
----------------

When Xorg is built without eudev, Xorg will be unable to automatically find and
use input devices. This requires the use of a "static" configuration using .conf
files.

NOTE: For this example I am configuring a touchpad alongside the keyboard. The
10-touchpad.conf file should be 1:1 transferable to a mouse's configuration.

NOTE: The below files should live in /etc/X11/xorg.conf.d/.

*   10-keyboard.conf

    The 10-keyboard.conf file is used to tell Xorg how it should setup the
    keyboard and which device node it should interact with.

         Section "InputDevice"
               Identifier "keyboard0"
               Option     "CoreKeyboard"
               Option     "AutoServerLayout" "true"

               # Use the libinput driver.
               Driver     "libinput"

               # This must point to the '/dev/input'
               # entry of your touchpad or mouse.
               Option     "Device" "/dev/input/event4"

               # Change the values of these to match
               # the detected layout of the command
               # 'setxkbmap -query'.
               Option     "XkbLayout" "us"
               Option     "XkbModel"  "pc105"
               Option     "XkbRules"  "evdev"
           EndSection

*   10-touchpad.conf

    The 10-touchpad.conf file is used to tell Xorg how it should setup the
    touchpad and which device node it should interact with.

          Section "InputDevice"
               Identifier "touchpad0"
               Option     "CorePointer"
               Option     "AutoServerLayout" "true"

               # Use the libinput driver.
               Driver     "libinput"

               # This must point to the '/dev/input'
               # entry of your touchpad or mouse.
               Option     "Device" "/dev/input/event5"

               # libinput options (optional).
               Option     "ScrollMethod" "twofinger"
               Option     "TappingDrag" "true"
               Option     "Tapping" "true"
           EndSection

Purging EUDEV
-------------

*   Disable the udevd service

        # From this point forward any device manager hotplugging functionality will be
        # unusable.
        $ unlink /var/service/udevd

*   Remove the eudev package

        # The removal is forced as the packages which depend on eudev will be rebuilt.
        $ KISS_FORCE=1 kiss r eudev

*   Generate a list of all packages which need to be rebuilt

        $ kiss-revdepends eudev
        libinput/depends:eudev
        xorg-server/depends:eudev

*   Rebuild all required packages

    NOTE: Some packages may have a mandatory dependency on eudev. You may receive
          errors when attempting to rebuild them. Simply re-install eudev until
          you are able to investigate further.

    NOTE: If the package manager pulls in eudev as part of the rebuild process,
          the package you are trying to rebuild has a mandatory dependency on
          eudev (and you cannot continue this exercise).

        $ kiss b libinput xorg-server

*   Verify that the eudev dependence is gone

        # The following command should output nothing. If it does not, the outputted
    packages require a rebuild.
        $ kiss-revdepends eudev

Changing Device Managers
------------------------

*   busybox mdev: Simply enable the `mdev` service.

        $ ln -s /etc/sv/mdev/ /var/service

*   Other device managers: Open an issue in $/kisslinux/init as the init scripts
    will need support for other device managers.

Reboot
------

If all went well, you should now be using mdev as your device manager while
retaining a fully working graphical environment. If input doesn't work under
Xorg, refer to the Xorg log file for information.
