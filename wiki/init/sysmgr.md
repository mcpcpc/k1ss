Managing services with SYSMGR
=============================

sysmgr is an alternative service supervisor written in POSIX sh. It is similar
in usage to runit.

Installation
------------

Begin by first verifying that you have sysmgr installed.

    # Available in the Community repository.
    $ kiss b sysmgr && kiss i sysmgr

Basic usage
-----------

As mentioned above, the usage of sysmgr is similar to runit.

| Action  | Command                                                            |
|---------|--------------------------------------------------------------------|
| List    | $ ls /etc/sysmgr/                                                  |
|         |                                                                    |
| Enable  | $ ln -s  /etc/sysmgr/SERVICE_NAME /var/sysmgr                      |
| Disable | $ unlink /var/sysmgr/SERVICE_NAME                                  |
|         |                                                                    |
| Stop    | $ svctl stop  SERVICE_NAME                                         |
| Start   | $ svctl start SERVICE_NAME                                         |

See svctl(1) for more usage information.

Running sysmgr on startup
-------------------------

sysmgr can be run at boot via /etc/inittab or a hook in /etc/rc.d.

    # Enabling on inittab
    ::respawn:/usr/bin/sysmgr

    # Enabling from /etc/rc.d/sysmgr.boot
    while :; do /usr/bin/sysmgr; done &

Switching from runit
--------------------

In order to switch from runit to sysmgr, copy the contents of the /var/service
directory to /var/sysmgr, and the same for /etc/sv to /etc/sysmgr.

    # Create the service directory for sysmgr
    $ mkdir -p /etc/sysmgr

    # Copy runit services
    for service in /etc/sv/*; do
        cp "$service/run" "/etc/sysmgr/${service##*/}"
    done

    # Copy all enabled services
    for service in /var/service/*; do
        ln -sf /etc/sysmgr/${service##*/} /var/sysmgr
    done
