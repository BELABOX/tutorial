Who should be using BELABOX today
=================================

BELABOX is experimental software and these instructions are intended for developers and advanced users who are willing to tinker, troubleshoot, submit patches, work out how to reproduce issues and record packet captures and logs when they run into problems.

If this doesn't sound like you, you should wait for a few more months until we can fix some of the known issues and develop some new features for a more consumer-friendly out-of-the-box experience.

**If someone is offering to sell a BELABOX-based streaming solution to you, don't buy it. The only reason we're not officially distributing a more consumer-friendly solution is because the software isn't quite ready for that yet. You'll also be stuck with unsupported software within a few months as there are a few tech stack changes planned for the near future.**

We're also prototyping some custom hardware which integrates everything together and improves the power efficiency of the now standard Jetson Nano + Cam Link setup. Only buy this hardware if you want to use it today.

If you want a consumer-friendly experience sooner, [please sign up to sponsor the development](https://github.com/sponsors/rationalsa). Virtually all development so far was unpaid, done as time permitted.


Intro
=====

You should consider joining [the Discord](https://discord.gg/hS8TcpsCKu) if you want to use BELABOX. We have a lot of other relevant information there and you can also get help if you're running into problems.

Check out [my Twitch channel](https://www.twitch.tv/rationalirl) for sample VODs streamed using BELABOX. If you're using BELABOX, please consider [sponsoring the ongoing development](https://github.com/sponsors/rationalsa).


Setting up BELABOX on a Jetson Nano
===================================

These instructions were written for L4T 32.4.4 (Based on Ubuntu 18.4 LTS) and last tested on Dec 20th, 2020. Consider this a temporary guide to setting up BELABOX while a more convenient solution is being developed.

Step -1
-------
You'll need another Internet-connected machine to serve as the ingest for your srtla-bonded stream. This can be a low cost VPS or a Linux computer (even a low power RPi) at home. Follow the *receiver* instructions in the [srtla readme](https://github.com/BELABOX/srtla/) for setting it up.

Step 1
------
Set up a micro SD card with L4T following the instructions from NVIDIA [for Jetson Nano 4GB](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) or [for Jetson Nano 2GB](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-2gb-devkit). Note that you can do the initial setup either using a monitor, mouse & keyboard or in headless mode using the USB serial console. For the rest of the tutorial we'll assume that the system was set up correctly and that you have SSH access to the Jetson Nano via the Ethernet network.

Step 2
------
Updating the system packages (this may take a while):

    sudo apt-get update
    sudo apt-get dist-upgrade

Step 3
------
Installing the required dependencies:

    sudo apt-get install nano build-essential git tcl libssl1.0-dev nodejs npm usb-modeswitch libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
    
Step 4
------
Add the google DNS servers so you have some resolvers accessible through any Internet-connected interfaces, as opposed to the servers accessible through a single mobile operator that you may be getting from DHCP:

    printf "\nnameserver 8.8.8.8\nnameserver 8.8.4.4\n" | sudo tee -a /etc/resolvconf/resolv.conf.d/head

Let's set up source routing for any wired modems. This uses a dhclient hook, which will execute for dhcp entries in `/etc/network/interfaces` or if called manually, but not for NetworkManager-managed devices. `/etc/network/interfaces` takes over configuring the ethX and usbX network interfaces from NetworkManager

    sudo wget https://raw.githubusercontent.com/BELABOX/tutorial/main/dhclient-source-routing -O /etc/dhcp/dhclient-exit-hooks.d/dhclient-source-routing
    sudo wget https://raw.githubusercontent.com/BELABOX/tutorial/main/interfaces -O /etc/network/interfaces
    printf "100 usb0\n101 usb1\n102 usb2\n103 usb3\n104 usb4\n" | sudo tee -a /etc/iproute2/rt_tables
    printf "110 eth0\n111 eth1\n112 eth2\n113 eth3\n114 eth4\n" | sudo tee -a /etc/iproute2/rt_tables

Also set up source routing for WiFi with NetworkManager:

    sudo wget https://raw.githubusercontent.com/BELABOX/tutorial/main/nm-source-routing -O /etc/NetworkManager/dispatcher.d/nm-source-routing
    sudo chmod 755 /etc/NetworkManager/dispatcher.d/nm-source-routing
    printf "120 wlan0\n121 wlan1\n122 wlan2\n123 wlan3\n124 wlan4\n" | sudo tee -a /etc/iproute2/rt_tables

If you have a WiFi adapter fitted, you can connect to a WiFi network with `sudo nmcli device wifi connect <AP NAME> password <WPA password>` after rebooting.

We use the `/etc/network/interfaces` configuration because it seems more reliable than Networkmanager at always bringing up all the interfaces. It also brings up all the modems even when they use the same MAC address (which is the case for several Huawei models), unlike NetworkManager.

Step 5
------
Disable the virtual Ethernet interface as it will cause naming conflicts if you use modems that get enumerated as `usbX` devices.

    sudo systemctl disable nv-l4t-usb-device-mode.service
    sudo reboot

Step 6
------
Installing (the BELABOX fork of) SRT:

    cd
    git clone https://github.com/BELABOX/srt.git
    cd srt
    ./configure --prefix=/usr/local
    make -j4
    sudo make install
    sudo ldconfig

Step 7
------
Building belacoder:

    cd
    git clone https://github.com/BELABOX/belacoder.git
    cd belacoder
    make
    
Step 8
------
Building srtla:

    cd
    git clone https://github.com/BELABOX/srtla.git
    cd srtla
    make

Step 9
------
Setting up belaUI:

    cd
    git clone https://github.com/BELABOX/belaUI.git
    cd belaUI
    git checkout ws_nodejs

Install the Node.js dependencies:

    npm install

Edit `setup.json` with the paths to the `belacoder` and `srtla` directories.

You can start the web interface to test it with:

    sudo nodejs belaUI.js

At this point BELABOX is ready to use, assuming that you have a capture card / other v4l2 input connected. Open `http://address_of_the_jetson` in a web browser. It will first ask you to set a password for access.

See the [belacoder readme](https://github.com/BELABOX/belacoder) for information about the available pipelines you can select in the *Encoder settings* menu of the web interface. Configure the *srtla settings* with the data for your ingest configured at *Step -1*.

After setting up and confirming that everything is working correctly, you can install belaUI as a system service that starts automatically at boot by running:

    sudo ./install_service.sh


Next steps
----------

For practical use, you should configure belaUI to be automatically started at boot and use a phone to control it.

If you become a recurring [github sponsor](https://github.com/sponsors/rationalsa), you'll get a BELABOX cloud remote account allowing you to manage your encoder from any Internet-connected device via [cloud.belabox.net](https://cloud.belabox.net/). Otherwise you'll need a direct LAN connection to your encoder to access belaUI. Depending on your modem setup, you could make belaUI accessible either through a modem that has both USB (for the Jetson) and WiFI (for the phone) interfaces, or by enabling the hotspot feature on the phone and connecting the Jetson to it as per step 4, or by setting up a Wifi access point on the Jetson Nano - outside the scope of this tutorial.


Receiving the stream
--------------------

Regardless of how many connections are available, BELABOX always streams via [srtla](https://github.com/BELABOX/srtla). To receive this stream, you have several options, including:

1) Become a [github sponsor](https://github.com/sponsors/rationalsa), support the BELABOX project and receive access to our hosted srtla/SRT relay service with servers in the US and France.
2) Follow the [srtla readme](https://github.com/BELABOX/srtla) to set up a basic relay using srt-live-transmit or another SRT server configured with the equivalent options.
3) Use a **third party** docker image configured to receive srtla, such as [this one](https://hub.docker.com/r/sherazarde/belabox-receiver). Note that we can make no guarantees about third party packages being maintained to support future revisions of the srtla software.


Ending notes
------------

If you're not confident following any of the instructions, please wait until we're able to distribute BELABOX in a more convenient format.

Once you're set up, check out [the bitrate guide](https://github.com/BELABOX/tutorial/blob/main/bitrate_guide.md).
