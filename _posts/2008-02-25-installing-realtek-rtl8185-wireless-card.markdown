---
layout: post
title: Installing a Realtek RTL8185 wireless card
summary: |
    In this post I explain how I made my wireless card with a Realtek
    RTL8185 chipset work in [Slackware GNU/Linux][].
    
    I recently moved to a new apartment and the Internet modem had to be
    placed in the bedroom, separate from the living room where I have my
    workstation, so I had to choose between buying a wireless card for my pc
    or drilling a hole through the wall to slip in an ethernet cable.
    
    Since I don't have a drill, and my wife wouldn't have liked a cable going
    through the middle of the makeup mirror, I got an Encore Electronics
    Wireless-G PCI Adapter ([ENLWI-G2][]). Actually this is just a card with a
    Realtek RTL8185 chipset on it, and you can get the Linux drivers from the
    [Realtek website][].
    
       [Slackware GNU/Linux]: http://www.slackware.com
       [ENLWI-G2]: http://www.encore-usa.com/product_item.php?region=us&bid=2&pgid=81_4&pid=285
       [Realtek website]: http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=1&PFid=1&Level=6&Conn=5&DownTypeID=3&GetDown=false&Downloads=true#RTL8185L
    
---

In this post I explain how I made my wireless card with a Realtek
RTL8185 chipset work in [Slackware GNU/Linux][].

I recently moved to a new apartment and the Internet modem had to be
placed in the bedroom, separate from the living room where I have my
workstation, so I had to choose between buying a wireless card for my pc
or drilling a hole through the wall to slip in an ethernet cable.

Since I don't have a drill, and my wife wouldn't have liked a cable going
through the middle of the makeup mirror, I got an Encore Electronics
Wireless-G PCI Adapter ([ENLWI-G2][]). Actually this is just a card with a
Realtek RTL8185 chipset on it, and you can get the Linux drivers from the
[Realtek website][].

   [Slackware GNU/Linux]: http://www.slackware.com
   [ENLWI-G2]: http://www.encore-usa.com/product_item.php?region=us&bid=2&pgid=81_4&pid=285
   [Realtek website]: http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=1&PFid=1&Level=6&Conn=5&DownTypeID=3&GetDown=false&Downloads=true#RTL8185L


Building the modules and testing
---------------------------------------------

I have to warn you that I had my PC crash a few times while
experimenting with the wireless settings, so be patient and be prepared to
hit the power button.

Download and extract the appropriate *\*.tar.gz* file from the Realtek
website, you will find instructions on how to build and load the modules
in the `readme` file. Basically you have to:

 1. Run the file `makedrv` to build the modules from the source code.

 2. Run the file `wlan0up` to load the modules into the running kernel.
     
    This is where I encountered the first problem, for some reason the
    last line in the `wlan0up` script (`ifconfig wlan0 up`) made my
    computer crash. After rebooting I commented out that line and tried
    again, no problem was found and I saw that the `wlan0` interface was
    already up anyway (run `ifconfig` to see the interfaces that are up).

    These modules are not going to be loaded automatically at boot up
    yet, we'll see how to do that later.

 3. The third step after the wireless interface is up is to configure your
    wireless link settings and getting an IP address, see the `readme`
    file to get the details.

In my case I only needed these two commands to set up the wireless link to
the router.

    iwconfig wlan0 essid "Megadeth"
    iwconfig wlan0 key abcd123456

My wireless LAN name is _"Megadeth"_ and it uses WEP encryption in Open
security mode. Yes, I know that WEP is practically useless for security,
but I have other devices that only talk WEP.

After setting the wireless link I had to run this command to get an ip
address from the router:

    dhcpcd -t 20 wlan0

I did *NOT* run the `wlan0dhcp` file that was provided in the tarball
as it appears to be RedHat specific, or something like that, I really
don't know, but I know for sure that it wouldn't work in Slackware.


Making the modules load at boot up
---------------------------------------------

The next step was to figure out how to have the modules loaded up when the
computer starts. 

One option is to leave the compiled modules where they are (or move it
wherever you want) and load them up by putting these instructions at the
end of the `/etc/rc.d/rc.modules-XYZ` script where `XYZ` is the version of
the kernel that the modules were built for:

    /sbin/insmod /root/rtl8185/ieee80211/ieee80211_crypt-rtl.ko
    /sbin/insmod /root/rtl8185/ieee80211/ieee80211_crypt_wep-rtl.ko
    /sbin/insmod /root/rtl8185/ieee80211/ieee80211_crypt_tkip-rtl.ko
    /sbin/insmod /root/rtl8185/ieee80211/ieee80211_crypt_ccmp-rtl.ko
    /sbin/insmod /root/rtl8185/ieee80211/ieee80211-rtl.ko
    /sbin/insmod /root/rtl8185/rtl8185/r8180.ko

The `/root/rtl8185/` folder is where I unpacked and built the modules.

The second option, which is the one I prefer, is to move the modules to
the appropriate kernel directories so that you don't clutter your root
folder.

But before that you should know that the ieee80211 modules that are built
from this package are intended as a replacement for the ieee80211 stack
that comes with the kernel. This means that you have to move out the
original ieee80211 stack to avoid having them loaded into the kernel. This
might not be strictly necessary since the modules have different names,
but I wasn't using the original stack and I didn't see any need for it so
I moved them out just as a precaution, as having both stacks loaded would
have caused a conflict.

    # Back up the original stack.
    cd /lib/modules/2.6.21.5-smp/kernel/net
    mv ieee80211 /root/ieee80211_original_stack

    # Move the new modules there.
    # we are at /lib/modules/2.6.21.5-smp/kernel/net
    mkdir ieee80211
    cp /root/rtl8185/ieee80211/*.ko ieee80211/

    # Finally move the r8180.ko module too.
    cd /lib/modules/2.6.21.5-smp/kernel/drivers/net/wireless
    cp /root/rtl8185/rtl8185/r8180.ko .

    # Update module dependencies.
    depmod -ae


The last command, `depmod -ae`, updates a file
`/lib/modules/2.6.21.5-smp/modules.dep` which indicates the dependencies
for each module. If you inspect that file you'll see that the `r8180`
module depends on the `ieee80211_rtl` and `ieee80211_crypt_rtl` modules;
indeed, if you reboot and type `lsmod` you'll see that these modules are
loaded.

    r8180
    ieee80211_rtl
    ieee80211_crypt_rtl

However, there are three additional modules that need to be loaded too:

    ieee80211_crypt_wep-rtl
    ieee80211_crypt_tkip-rtl
    ieee80211_crypt_ccmp-rtl

Without these modules you won't be able to set encryption keys for your
wireless link. These are not being loaded because no module depends on
them. To change that we have to edit the `modules.dep` file and edit the
line that specifies the dependencies of the `r8180` module, the format of
a `modules.dep` entry is:

    /path/to/module_a.ko: /path/to/module2.ko /path/to/module1.ko

Which means that `module_a.ko` depends on `module1.ko` and `module2.ko`.
The modules are loaded from right to left, which means that module1 is
loaded first, followed by module2 and finally module\_a last.

So then, open up the `modules.dep` file and search for the line where the
dependencies of the `r8180` are defined and change it so that all the
necessary modules are loaded.

Here I break the line into multiple lines so that you can read it easily,
but make sure it is only *one whole line*, so please pay attention. Also,
the order is important, so pay double attention.

I repeat, this *must be one single line* in your `modules.dep` file:

    /lib/modules/2.6.21.5-smp/kernel/drivers/net/wireless/r8180.ko:
    /lib/modules/2.6.21.5-smp/kernel/net/ieee80211/ieee80211-rtl.ko
    /lib/modules/2.6.21.5-smp/kernel/net/ieee80211/ieee80211_crypt_ccmp-rtl.ko
    /lib/modules/2.6.21.5-smp/kernel/net/ieee80211/ieee80211_crypt_tkip-rtl.ko
    /lib/modules/2.6.21.5-smp/kernel/net/ieee80211/ieee80211_crypt_wep-rtl.ko
    /lib/modules/2.6.21.5-smp/kernel/net/ieee80211/ieee80211_crypt-rtl.ko

After you reboot you'll see that all the modules have been loaded up.


Setting the wireless link and getting an IP at boot up
---------------------------------------------

The final step is to set your wireless *essid* and *key* and get an ip
from the router when your computer boots up.

Since I run Slackware, the place to define ip settings is the file
`/etc/rc.d/rc.inet1.conf` - the relevant lines in my settings file are:

    ...
    IFNAME[4]="wlan0"
    USE_DHCP[4]="yes"
    DHCP_TIMEOUT[4]=30
    WLAN_ESSID[4]=Megadeth
    WLAN_KEY[4]="abcd123456 open"
    ...

There are a few things to notice here.

I added the variable `DHCP_TIMEOUT`, this is because the
`/etc/rc.d/rc.inet1` script executes an `ifconfig wlan0 up` if that
variable is not defined (look around line 117), and I don't know why, but
for some reason sometimes that makes my computer freeze.

This is strange because the `rc.wireless` script also executes this
command but I have not found any problem with it, it seems that the card
doesn't like to be "upped" so many times.

For the `WLAN_ESSID` variable you'll type your wireless LAN name without
quotes.

You'll notice that the `WLAN_KEY` contains the word *open*, this is
because if you don't specify either *open* or *restricted* the
`/etc/rc.d/rc.wireless` script will set the key to *restricted* mode by
default. In my case I wanted it to be in *open* security mode.

One last thing that is important, the `/etc/rc.d/rc.wireless` script sets
a *nick* option on the wireless settings, but this driver does not support
that option and it will throw an error. Open that file and comment out
that line, look around line 184, i.e.:

    if [ ! -n "$NICKNAME" ] ; then
        NICKNAME=`/bin/hostname`
    fi
    if [ -n "$ESSID" -o -n "$MODE" ] ; then
        echo "$0:  $IWCOMMAND nick $NICKNAME" | $LOGGER
        # $IWCOMMAND nick $NICKNAME <-- this is not supported
    fi

And that's it, try running `/etc/rc.d/rc.inet1 stop` and then
`/etc/rc.d/rc.inet1 start`.


You'll notice that this line is printed on the screen:

    ./rc.wireless:  wlan0 information: 'Any ESSID...'

If it bothers you, you can edit the `/etc/rc.d/rc.wireless.conf` file and
comment out the lines from line number 38 to 41 and you won't see that
message anymore.

Or, you can change the `INFO` variable to something else. I'm more
nostalgic and so I changed it to a quote from an old song I like:

    ## --------- START SECTION TO REMOVE -----------
    ## Pick up any Access Point, should work on most 802.11 cards
    *)
        INFO="The warheads will all rust in peace"
        ESSID="Megadeth"
        ;;
    ## ---------- END SECTION TO REMOVE ------------

It's not necessary to put the `ESSID` here as it is overridden by what you
set on the `rc.inet1.conf` file.


If things get really nasty....
---------------------------------------------

If you screwed it with the wireless settings and you can't get your pc to
boot up normally because the driver crashes it, you can boot up in single
user mode (also called emergency mode).  When you boot up your computer
and get the LILO prompt, select your Linux installation and type the word
*single* after the label.

In single user mode no networking configurations will be set up at all so
that you can edit your settings and try again.

Enjoy...




<!-- vim: set tw=74 sw=4 ts=4 et spell filetype=mkd: -->
