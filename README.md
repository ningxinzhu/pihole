#Installing and configuring Pi-hole v2.10 on the Raspberry Pi 3

Written 22 Dec 2016 by @ningxinzhu ... though you can find most of this on [the Pi-hole FAQ, especially its page about pihole commands](https://discourse.pi-hole.net/t/the-pihole-command-with-examples/738), /r/raspberry_pi, and /r/pihole. Thank God for good documentation and reddit.

[My Pi-hole web interface](https://i.imgur.com/NJ3v4TT.jpg)

##What you will need
- Raspberry Pi 3 and its power supply
  - I put a case and two heatsinks on my Pi in case it gets too hot.
- microSD card
  - Raspbian takes up almost 4 GB so although a 4 GB microSD card will work, 8 GB will give you more peace of mind that Pi-hole can do what it needs to do.
- an adapter so that you can format said microSD card to FAT32 using a computer
- Ethernet cable
- USB keyboard and mouse
  - If it's not a QWERTY keyboard with **all** of the keys normally found on a keyboard, Pi-hole can still be installed but it will take much longer. I learned this the hard way.
- HDMI monitor
- a router to which you have admin access

##Installation
- Format the microSD card to FAT32 using [SDFormatter](https://www.sdcard.org/downloads/formatter_4/), Disk Utility on OS X, or gparted on Linux.
- Download [NOOBS](https://www.raspberrypi.org/downloads/noobs/) and unzip it to the root of your microSD card.
- Insert the microSD card into the Pi. Connect the Pi to your router using an Ethernet cable. Connect the keyboard/mouse/monitor to the Pi.
- Connect the Pi with a power outlet using the power supply; the Pi will now boot. Install Raspbian (Jessie, not Lite) by following the on-screen instructions.
- Run `git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole` in Terminal to clone the Pi-hole repository on Github to your Pi.
- Navigate to /home/pi/Pi-hole/automated install/ in File Manager and copy the script basic-install.sh to /home/pi/ (or any other location that is easy to type into Terminal).
- Run `bash /home/pi/basic-install.sh` in Terminal; the Pi-hole installer will now run.
  - 'Do you want to use your current network settings as a static address?' YES.
  - Remember to enable IPv6!
- When the installation is complete, copy down the IPv4 and IPv6 addresses that you will direct your router to use as a DNS server. Also copy down the address of the Pi-hole web interface and the currently set password.
- Log into your router's admin dashboard (default directions to which are often found on the bottom of a router) and modify the DNS settings so that it uses your Pi-hole as a DNS server.
  - I did not set an alternate DNS server but you can use 8.8.8.8 which is Google's.
  - Some routers need a restart after changing the DNS settings.
- Run `pihole enable` and then `pihole status` to make sure DNS blocking is working. You can also see that it is working from the web interface.
- You need the password which was setup during installation to do anything besides view the basic DNS blocking statistics in the web interface, but you can change that password by running `pihole -a -p TYPE_YOUR_NEW_PASSWORD_HERE_INSTEAD_OF_THIS_PLACEHOLDER_TEXT` in Terminal. Running just `pihole -a -p` removes the password entirely and anyone on the network can modify Pi-hole if they know the address of the web interface.

##Configuration of adlists and whitelist
- Run `sudo cp /etc/pihole/adlists.default /etc/pihole/adlists.list` in Terminal. What this does is it copies the contents of the file adlists.default into a new file called adlists.list, which are both located in /etc/pihole/.
- Run `sudo nano /etc/pihole/adlists.list` so that you can add the following ad-serving domains:
```
https://raw.githubusercontent.com/AdAway/adaway.github.io/master/hosts.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_malvertising.txt
https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/hosts.txt
https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/dom-bl-base.txt
https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/Mandiant_APT1_Report_Appendix_D.txt
https://hosts-file.net/emd.txt
https://hosts-file.net/exp.txt
http://www.hostsfile.org/Downloads/hosts.txt
https://raw.githubusercontent.com/joeylane/hosts/master/hosts
http://winhelp2002.mvps.org/hosts.txt
https://adblock.mahakala.is/
https://mirror.cedia.org.ec/malwaredomains/immortal_domains.txt
https://www.malwaredomainlist.com/hostslist/hosts.txt
https://mirror.cedia.org.ec/malwaredomains/justdomains
http://hostsfile.mine.nu/hosts0.txt
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml
https://raw.githubusercontent.com/piwik/referrer-spam-blacklist/master/spammers.txt
https://raw.githubusercontent.com/quidsup/notrack/master/malicious-sites.txt
https://raw.githubusercontent.com/quidsup/notrack/master/trackers.txt
https://raw.githubusercontent.com/ReddestDream/reddestdream.github.io/master/Projects/MinimalHosts/etc/MinimalHostsBlocker/minimalhosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.2o7Net/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.Dead/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/KADhosts/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.Risk/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.Spam/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/Telemetry/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/tyzbit/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/UncheckyAds/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/SpotifyAds/hosts
http://someonewhocares.org/hosts/zero/hosts
https://raw.githubusercontent.com/vokins/yhosts/master/hosts
https://raw.githubusercontent.com/eladkarako/hosts.eladkarako.com/master/_raw__hosts.txt
```
- CTRL-O to save the file; nano also asks you the filepath to save it to (leave it the same).
- Run `sudo nano /etc/pihole/whitelist.txt` so that you can [add domains to the whitelist](https://github.com/ningxinzhu/pihole/blob/master/whitelist.txt).
  - The good news is that you can easily add domains to the whitelist in the Pi-hole web interface, but you can't add more than one domain at once there, so it's good to know how to do it in Terminal (or through PuTTY / SSH) for the initial configuration.
- Again, CTRL-O to save the file; nano also asks you the filepath to save it to (leave it the same).
- Remove the adlists.default file that you copied earlier by running `sudo rm /etc/pihole/adlists.default`. It's okay because you just made a much more comprehensive adlists file that includes all of the information in the one you just deleted.
- Run `pihole -g` to update Pi-hole with the new list of ad-serving domains and the new whitelist. Some of these adlists are gigantic so don't be alarmed if Terminal doesn't seem to display any activity at times - things are still being updated.
- The web interface should now show an absolutely massive number of domains being blocked (mine is currently 923673). You can also whitelist any domain which has accidentally been blocked; for instance, I whitelisted mp.weixin.qq.com because websites that I accessed from within WeChat would not load.
- Optional: in the web interface, you can add these domains to the blacklist in order to try and block ads on YouTube, but YMMV (it didn't block any YouTube ads for me):
```
r4---sn-vgqs7nez.googlevideo.com
r4.sn-vgqs7nez.googlevideo.com
www.youtube-nocookie.com
i1.ytimg.com
r17---sn-vgqsenes.googlevideo.com
r2---sn-vgqs7n7k.googlevideo.com
r1---sn-vgqsen7z.googlevideo.com
r1.sn-vgqsen7z.googlevideo.com
r20---sn-vgqs7ne7.googlevideo.com
r20.sn-vgqs7ne7.googlevideo.com
```
