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

## "Fix" Recurring Ubuntu Plymouth Error on Startup
On startup, ubunutu calls the plymouth deamon.
This handles the use of splash screens for boot and shutdown, but is known to be buggy in ubuntu 16.04 gnome.

The problem can be fixed by installing the full plymouth package via the command:
```
sudo apt install plymouth-x11
```
A reboot will then confirm the correct function of the implemented fix by not producing an error.

## Grub Improvements
A few changes to the default grub bootloader setup can make it more functional and visually attractive.

To change the settings such that the bootloader will wait for user input before automatically selecting the default option run the following commands to edit the grub config file.
```
cd /etc/default/
ls
sudo gedit grub
```
This will change to the location of grub's config file and open it in the gedit text editor with write privileges.

It can then be altered to match the following code.
__Note:__ the `GRUB_THEME` line should be uncommented only once the theme has been installed.
```
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=2
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=-1
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
GRUB_GFXMODE=1920x1080
#GRUB_THEME="/boot/grub/themes/Vimix/theme.txt"
# Theme copied from https://github.com/vinceliuice/grub2-themes

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
```
The most relevant parameters here are the:
  * `GRUB_TIMEOUT_STYLE=menu` which will apply the timeout period once the main screen has been displayed.
  * `GRUB_TIMEOUT=-1` which sets the time before the bootloader automatically selects an OS to infinite.
  * `GRUB_GFXMODE=1920x1080` which enables the graphics mode options in grub and sets the resolution to the of a standard HD monitor.
  * `GRUB_THEME=<path>\theme.txt` which informs grub that it should use a theme document and provides the location of a plaintext descriptor for such.
  
The file can now be saved and closed.

Run the folowing to inform grub of the updates and reboot to see their effects.
```
sudo update-grub
```

### Add a Theme
To improve the visual appeal of the grub loader it is also possible to add a visual theme such as [Vimix](https://github.com/vinceliuice/grub2-themes).

This is carried out by first copying the theme files to the downloads folder via:
```
git clone https://github.com/vinceliuice/grub2-themes.git
```
The theme may work from here, but its best to move it somewhere it's less linkely to be disturbed.
One option is to store in alongside the other grub config files, making the following directory.
```
sudo mkdir /boot/grub/themes
```
However, care should always be taken when altering the contents of `/boot/`.

The images and descriptor for the theme can then be copied into this directory via
```
sudo cp -r ~/Downloads/grub2-themes/grub-themes-vimix/Vimix ~/boot/grub/themes/
```
Once this is complete the downloaded theme can be removed via the below, cleaning up the downloads folder.
```
rm -rdf grub2-themes/
```
The final step is then to uncomment the reference to `GRUB_THEME` above and update grub.
The changes will then be visible on reboot.

### Improve the theme
The background to the theme is jpeg-y and unatttractive, it can be replaced with [another option](background.png) by copying the file into the directory created above and altering the the `themes.txt` file to match the folowing:

```
# GRUB2 gfxmenu Linux Vimix theme
# Designed for any resolution

# Global Property
title-text: ""
desktop-image: "background.png"
desktop-color: "#000000"
terminal-font: "Ubuntu R *.pf2"
terminal-box: "terminal_box_*.png"
terminal-left: "0"
terminal-top: "0"
terminal-width: "100%"
terminal-height: "100%"
terminal-border: "0"

# Show the boot menu
+ boot_menu {
  left = 60%
  top = 20%
  width = 30%
  height = 60%
  item_font = "Ubuntu R 16"
  item_color = "#cccccc"
  selected_item_color = "#ffffff"
  item_height = 24
  item_spacing = 12
  selected_item_pixmap_style = "select_*.png"
}

# Show a countdown message using the label component
+ label {
  top = 82%
  left = 35%
  width = 30%
  align = "center"
  id = "__timeout__"
  text = "Booting in %d seconds"
  color = "#cccccc"
  font = "Ubuntu R 16"
}

# Show a message with instruction prompt
+ label {
  left = 60%
  top = 18%
  width = 30%
  height = 10%
  font = "Ubuntu R 36"
  align = "left"
  text = "Select Boot Option:"
  color = "#cccccc"

}
```
__Note:__ The font sytle options appear not to have any effect so the relevent .pf2 files are not provided.

As before, the changes are visible upon update of grep and reboot.
