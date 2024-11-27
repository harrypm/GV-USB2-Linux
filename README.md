GV-USB2 Linux Driver
====================

A linux driver for the IO-DATA GV-USB2 SD capture device.


# Streaming on Linux

## Goal

* User GV-USB2 Capture Card
* Use LiveSplit
* Show Input Display

## GV-USB2 Driver

1. Clone the [Linux Driver Repository](https://github.com/Isaac-Lozano/GV-USB2-Driver) (by Isaac-Lozano)
2. In the new Folder run the command `make`. This will build the driver. If some dependencies are missing try to run `sudo apt update && sudo apt install build-essential linux-headers-$(uname -r)` or an equivalent command on your distribution first.
3. To activate the drivers you can use `insmod`:
    3.1. The sound driver can cause an issue of not finding the correct index to mount new devices. Therefore run the command `cat /proc/asound/cards` and note the highest device number seen.
    3.2. To activate the sound driver run `sudo insmod gvusb2-sound.ko index=NEW_INDEX` while replacing `NEW_INDEX` with the highest device number from the previous command plus one.
    3.3. To activate the video driver run `sudo insmod gvusb2-video.ko`.
4. There might be an issue of OBS crashing when turning on the captured device while the app is already running. Therefore turn on your console and connect everything before launching OBS to test the drivers.

## Automatically load the drivers

```sh
#!/bin/bash

# Compile Kernel Modules
make clean
make

# Move to kernel path
sudo rm -f /lib/modules/$(uname -r)/drivers/video/gvusb2-*.ko
sudo cp *.ko /lib/modules/$(uname -r)/drivers/video

# Register modules
sudo depmod

# Find index for the audio module
local alsa_index=$(cat /proc/asound/cards | grep ]: | wc -l)

# Write auto-load configuration
sudo echo "usbtv
gvusb2-sound index=$alsa_index
gvusb2-video" > /etc/modprobe.d/gvusb.conf
```

## OBS Launcher (Legacy)

I like to use a script to launch OBS, that also activates the GV-USB2 drivers. I also added an alert box before launching OBS to remind me of starting my console first (feel free to customize the text).

Create new files called `init.sh` and `launch-obs.sh` in your home directory and add the following lines

```sh
#!/bin/bash
insmod /home/scaramanga/gvusb/gvusb2-sound.ko index=<Sound driver index>
insmod /home/scaramanga/gvusb/gvusb2-video.ko
```

```sh
#!/bin/bash
sudo ./init.sh > /dev/null 2>&1
zenity --info --text="Make sure the capture card is plugged in and the Wii is running!"
obs > /dev/null 2>&1 &

```

Afterwards run `chmod +x init.sh launch-obs.sh` to make the scripts executable.

### Desktop Icon

To add a launcher icon to your desktop create a file called `OBS.desktop` on your desktop. Add the following lines to it

```
#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Exec=/path/to/launch-obs.sh
Name=OBS Studio
Comment=OBS + Init
Icon=/usr/share/icons/hicolor/256x256/apps/com.obsproject.Studio.png
```

Replace the path in the line starting with `Exec` with the absolute path to your OBS launch script. Make the file executable by running `chmod +x OBS.desktop`. Then right-click the file and select the option "Allow Launching". Now you can start OBS by double clicking the file.

If the OBS icon is not shown you might have to search for it on your file system and replace the path in the line starting with `Icon`.

### Skip authenticating for running the init script

> :warning: This step is not recommended if you use your Linux installation for anything but streaming, since it allows to run a script with root priviliges without entering a password :warning:

Open the sudoers config using your preferred editor by typing `sudo visudo` into your terminal. At the bottom of the document add the line
```
<username> ALL=(root) NOPASSWD: /path/to/init.sh
```

replacing `<username>` with your username and the path by the absolute path to your `init.sh`.

## Install LiveSplit

To run LiveSplit on Linux you need to install `wine` and .NET 4. To do so run
```sh
sudo apt update && sudo apt install -y wine winetricks && winetricks dotnet48 && winetricks allfonts
```

At some point an installation dialog for .NET will appear, which you have to click through. This step will take quite some time.

Afterwards you can run LiveSplit using the command `wine /path/to/LiveSplit.exe`.

To install the font `Segoe UI` used in the default layout, follw the [instructions here](https://needforbits.wordpress.com/2017/07/19/install-microsoft-windows-fonts-on-ubuntu-the-ultimate-guide/).

### Synchronized LiveSplit Installation using Dropbox

I like to have my LiveSplit folder on Dropbox. That way all my Settings (including used Layouts and Split files) are synced accross machines. To link the Dropbox folder of your Linux machine to `wine` you can create a symlink.

By default the Windows file system of wine is mirrored under `~/.wine/drive_c`. Here navigate to the folder in which your Dropbox-Folder on windows resides e.g. `~/.wine/drive_c/users/<WindowsUsername>`. Now run the command `ln -s ~/Dropbox/ Dropbox`. That creates a symbolic reference to your Linux-Dropbox folder.

### Desktop Icon

Like with OBS create an executable `*.desktop`-file for LiveSplit with the following content

```
#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Exec=/path/to/launch-livesplit.sh
Name=LiveSplit
Comment=
Icon=/path/to/livesplit.png
```

The script `launch-livesplit.sh` must be executable and contain the following lines

```sh
#!/bin/bash
wine ~/Dropbox/LiveSplit/LiveSplit/LiveSplit.exe &
```

You can find the icon `livesplit.png` online easily.

### Caveats

In some parts of LiveSplit dots and colons are not rendered. Most of the time you won't be able to change fonts in the Layout editor.

### LiveSplit One

It is worth to follow the development of LiveSplit One, a multiplatform version of LiveSplit. At the moment, a few key features like the WR component are still missing and there are no official native builds. However a working online version can be found at [https://one.livesplit.org/](https://one.livesplit.org/).

## Input Display

The most common application offering an input display is NintendoSpy. Instead of trying to emulate it with `wine` (which might be feasable), I went for the multiplatform alternative *Open Joystick Display*. It offers full NintendoSpy support and also works with way more controllers. However it is a bigger and heavier application, so I personally wouldn't use it on Windows.

To be able to read the controller signal you ahve to launch the application as root. I set up a desktop icon running a script, which in turn runs another script with root privileges. For that second script I added another exception in the sudoers file, so I'm not asked for a password (again, this is not secure!).

OJD comes with themes for many controllers. For my own use, I ported [Aliens'](https://twitter.com/aliensqueakytoy) populat Tron GameCube theme to the OJD format, you can find the port [here](https://github.com/scaramangado/ojd-tron-theme).

To install the theme download and extract the folder. Then rename it to `ojd-tron-theme`, afterwards you can select its parent folder as your theme root in OJD.
