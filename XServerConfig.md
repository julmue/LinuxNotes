# XServer

What is the X-server?

## Xinerama

Xinerama is an extension to X window System that enables X application
and window managers to use two or more phyiscal displays as one large
display

## On how many billion, trillion places do I configure X-server?

Only for global changes that affect all users
`/etc/X11/xorg` conf gets read only at the start of the x-server.
the files in the folders `/usr/lib/X11/xorg.con.d/`
and/or `/usr/share/X11/xorg.cond.d/` get read during every plugin of a new device
there are some differences to `etc/X11/xorg.conf`.

By default debian based systems have no `/etc/X11/xorg.conf`.

to creage one:
1. go to console
2. stop window manager (sudo service mdm stop,..)
3. `Xorg -configure`
   this will create `xorg.conf.new` in Home (with some insignificant error messages)
4. restart window manager (`sudo service mdm start`)
5. open home folder and edit
6. `sudo cp xorg.conf /etc/X11`

#### /etc/X11/xorg.conf

    man xorg.conf

to use multi-head:
USE THE XRANDR SYNTAX FOR GOOOOOOOODS SAKE!!!!!!!!!!!
PLEASE,SO SIMPLE ... XRANDR XORG.CONF!!!
