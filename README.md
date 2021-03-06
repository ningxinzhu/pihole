# Installing and configuring Pi-hole v2.10 on the Raspberry Pi 3

Written 24 Dec 2016 by @ningxinzhu ... though you can find most of this on [the Pi-hole FAQ, especially its page about pihole commands](https://discourse.pi-hole.net/t/the-pihole-command-with-examples/738), /r/raspberry_pi, and /r/pihole. Thank God for good documentation and reddit.

[My Pi-hole web interface](https://i.imgur.com/NJ3v4TT.jpg)

## What you will need
- Raspberry Pi 3 and its power supply
  - I put a case and two heatsinks on my Pi in case it gets too hot.
- microSD card
  - Raspbian Jessie takes up almost 4 GB so although a 4 GB microSD card will work, 8 GB will give you more peace of mind that Pi-hole can do what it needs to do.
- an adapter so that you can format said microSD card to FAT using a computer
- Ethernet cable
- **full-size** USB keyboard and USB mouse, or USB mouse plus SSH and VNC
- HDMI monitor
- a router to which you have admin access

I would strongly recommend having the following applications installed on your computer as well. If you enable SSH and VNC as soon as Raspbian Jessie is installed, and you have these applications set up, you can control the Pi remotely using the keyboard and mouse on your computer. The Raspberry Pi Foundation has excellent documentation about [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/) and [VNC](https://www.raspberrypi.org/documentation/remote-access/vnc/).
- [Nmap](https://nmap.org/download.html), to determine the IP address of and open ports on your Pi
- [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html), an SSH client
- [RealVNC VNC Viewer](https://www.realvnc.com/download/viewer/)

## Installation
- Format the microSD card to FAT using [SDFormatter](https://www.sdcard.org/downloads/formatter_4/), Disk Utility on OS X, or gparted on Linux.
- Download [the offline NOOBS installer](https://www.raspberrypi.org/downloads/noobs/) and unzip it to the root of your microSD card.
- Insert the microSD card into the Pi. Connect the Pi to your router using an Ethernet cable. Connect the keyboard/mouse/monitor to the Pi.
- Connect the Pi with a power outlet using the power supply; this should always be the last peripheral to be connected because the Pi will now boot. Install Raspbian (Jessie, not Lite) by following the on-screen instructions.
  - Lite does not have a desktop environment; consequently, getting the Pi to boot straight into the desktop logged in as pi@raspberrypi requires additional work.
- Enable SSH and VNC in the start menu > Preferences > Raspberry Pi Configuration > Interfaces.
  - Usually this is when I switch over to to VNC for the rest of the installation and configuration. Use Nmap to figure out the IP address of the Pi.
- The simplest method of installing Pi-hole is to run `curl -sSL https://install.pi-hole.net | bash` in Terminal, but doing this is the equivalent of running an .exe file of unknown origin on Windows and giving it administrator access. I usually choose to clone the Pi-hole repository using git, have a look at what's inside, and then run the installer script myself. Do this by running `git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole` in Terminal.
- Navigate to /home/pi/Pi-hole/automated install/ in File Manager and copy the script basic-install.sh to /home/pi/ (or any other location that is easy to type into Terminal).
- Run `sudo bash /home/pi/basic-install.sh` in Terminal; the Pi-hole installer will now run.
  - 'Do you want to use your current network settings as a static address?' YES.
  - Remember to enable IPv6!
- When the installation is complete, copy down the IPv4 and IPv6 addresses that you will direct your router to use as a DNS server. Also copy down the address of the Pi-hole web interface and the currently set password.
- Log into your router's admin dashboard (default directions to which are often found on the bottom of a router) and modify the DNS settings so that it uses your Pi-hole as a DNS server.
  - I did not set an alternate DNS server but you can use 8.8.8.8 and 8.8.4.4 which are Google's.
  - Some routers need a restart after changing the DNS settings.
- Run `pihole enable` and then `pihole status` to make sure DNS blocking is working. You can also see that it is working from the web interface.
- You need the password which was setup during installation to do anything besides view the basic DNS blocking statistics in the web interface, but you can change that password by running `pihole -a -p TYPE_YOUR_NEW_PASSWORD_HERE_INSTEAD_OF_THIS_PLACEHOLDER_TEXT` in Terminal. Running just `pihole -a -p` removes the password entirely and anyone on the network can modify Pi-hole if they know the address of the web interface.

## Configuration of adlists, whitelist, blacklist
- Run `sudo cp /etc/pihole/adlists.default /etc/pihole/adlists.list` in Terminal. What this does is it copies the contents of the file adlists.default into a new file called adlists.list, which are both located in /etc/pihole/.
- Run `sudo nano /etc/pihole/adlists.list` so that you can add the following ad-serving domains:
```
https://dbl.oisd.nl/
https://porn.oisd.nl
```
- CTRL-O to save the file; nano also asks you the filepath to save it to (leave it the same).
- Run `sudo nano /etc/pihole/whitelist.txt` so that you can [add domains to the whitelist](https://github.com/ningxinzhu/pihole/blob/master/whitelist.txt). Do the same for the [blacklist](https://github.com/ningxinzhu/pihole/blob/master/blacklist.txt).
  - The good news is that you can easily add domains to the whitelist/blacklist in the Pi-hole web interface, but you can't add more than one domain at once there, so it's good to know how to do it in Terminal (or through PuTTY / SSH) for the initial configuration.
- Again, CTRL-O to save the file; nano also asks you the filepath to save it to (leave it the same).
- Remove the adlists.default file that you copied earlier by running `sudo rm /etc/pihole/adlists.default`. It's okay because you just made a much more comprehensive adlists file that includes all of the information in the one you just deleted.
- Run `pihole -g` to update Pi-hole with the new list of ad-serving domains and the new whitelist/blacklist. Some of these adlists are gigantic so don't be alarmed if Terminal doesn't seem to display any activity at times - things are still being updated.
- The web interface should now show an absolutely massive number of domains being blocked.
