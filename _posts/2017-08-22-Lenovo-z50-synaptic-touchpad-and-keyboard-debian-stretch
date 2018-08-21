Following the stable release of Debian Stretch (and after coming back off holiday) I have re-installed my laptop replacing KDE Neon with Debian 9 and well KDE. I can't speak highly enough about KDE and it has recently become the GUI of choice for me.

It is also nice to finally come off Ubuntu 16.04, which does seem to have the best support out there, but can/has been unreliable, with issues reading gpg secret keys and every so often not bring up a dhcp IP address.

But there was one issue that I came across with the tracker pad not working, while typing on the keyboard. Which is an issue you are unlikely to come across until you start playing games. However there is a way to disable this feature, which involves going to the start menu typing in touchpad and opening the setting and going to the Enable/Disable Touchpad tab and untick Disable touchpad when typing. Or so you would think. But no, no luck the touchpad is still being disabled.

First thing to check is all the drivers are installed. So to find out what drivers we need
///
lspci | grep -i touchpad
lsusb | grep -i touchpad
dmesg | grep -i touchpad
///
This tells you what make and model we are dealing with, which turns out to be
///
input: SynPS/2 Synaptics TouchPad as /devices/platform/i8042/serio1/input/input2
///
Now with this information we can check online, which takes me to this wiki page https://wiki.debian.org/SynapticsTouchpad

But this doesn't have the information I need and doesn't seem to be for Debian Stretch. If in doubt check the installed documentation
///
liam@home:~$ cd /usr/share/doc
liam@home:/usr/share/doc$ grep -R synaptics *
///
Where we find
///
/usr/share/doc/xorg/howto/configure-input.html
///
So we open this in firefox and we find on the page:
///
synaptics configuration
///
So we install this example configuration from this page
///
Section "InputClass"
    Identifier "touchpad tweaked catchall"
    MatchIsTouchpad "on"
    Driver "synaptics"
    Option "TapButton1" "1"
    Option "HorizEdgeScroll" "1"
EndSection
///
Reboot the laptop and there we have it, I can hold the arrow keys down as long as I want and use the Touchpad.
