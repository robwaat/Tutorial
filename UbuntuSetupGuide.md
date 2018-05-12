# Ubunutu Setup

A brief note of steps used to setup my ubuntu / linux work stations

## Seting the ntp server to work with Strathclyde Eduroam
Run the command below to open the `timesyncd.conf` file
```
sudo gedit /etc/systemd/timesyncd.conf
```

Edit this file to resemble the folowing
```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See timesyncd.conf(5) for details.

[Time]
NTP=ntp.cis.strath.ac.uk 0.uk.pool.ntp.org 1.uk.pool.ntp.org 2.uk.pool.ntp.org 3.uk.pool.ntp.org
#FallbackNTP=ntp.ubuntu.com
```
Note the uncommented line commencing NTP. This sets the strath NTP server as the primary source of time synchronisation while also using the uk ntp pool servers as backup in the event strath is unavailable.

Save the file and reboot

Operational status may then be confirmed via
```
systemctl status systemd-timesyncd.service
```

## Setup trackpad on Dell Prcision 5520
The default installation of Ubuntu 16.04 Gnome has trouble recognising the touchpad and adjusting its setting regarding scrolling direction and tap-to-click.
Changing to the `libinput` driver circumvents this.

First, install the new driver package with
```
sudo apt install xserver-xorg-input-libinput-hwe-16.04
```
Then look for the old synaptic driver with
```
apt list --installed | grep xorg
```
It should be something like `xserver-xorg-input-synaptics-hwe-16.04`, which must be completely removed in order for the new driver to function.

The old driver can now be remove via the command
```
sudo apt-get purge xserver-xorg-input-synaptics-hwe-16.04
```
Once this is complete, reboot the pc and alter the settings via the standard mouse / touchpad gui.

Enable the `native scrolling` and `tap-to-click` options.

It may then also be desirable to add gesture support via libinput-gestures.
