Update: while these instructions are still current, you should consider joining [the Discord](https://discord.gg/W9bD9YUC6E) if you want to use BELABOX. We have a lot of other relevant information there and you can also get help if you're running into problems.

Check out [my Twitch channel](https://www.twitch.tv/rational_sail) for sample VODs streamed using BELABOX. If you're using BELABOX, please consider [sponsoring the ongoing development](https://github.com/sponsors/rationalsa).


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

    sudo apt-get install nano build-essential git tcl libssl-dev ruby ruby-sinatra ruby-sinatra-contrib usb-modeswitch libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
    
Step 4
------
Set up source routing. Note that `/etc/network/interfaces` will take over configuring the ethX and usbX network interfaces from NetworkManager:

    sudo wget https://gist.github.com/rationalsa/57657d36ff13582e5b309815fa32cd63/raw/6986fcf64ea8ad57fc4b3cb4e585e96d64e4dfec/if-post-up-source-route.sh -O /opt/if-post-up-source-route.sh
    sudo wget https://raw.githubusercontent.com/BELABOX/tutorial/main/interfaces -O /etc/network/interfaces
    sudo chmod 644 /etc/network/interfaces
    sudo chmod 755 /opt/if-post-up-source-route.sh
    printf "100 usb0\n101 usb1\n102 usb2\n103 usb3\n104 usb4\n" | sudo tee -a /etc/iproute2/rt_tables
    printf "110 eth0\n111 eth1\n112 eth2\n113 eth3\n114 eth4\n" | sudo tee -a /etc/iproute2/rt_tables
    sudo reboot

Step 4
------
Installing (the BELABOX fork of) SRT:

    cd
    git clone https://github.com/BELABOX/srt.git
    cd srt
    ./configure --prefix=/usr/local
    make -j4
    sudo make install
    sudo ldconfig

Step 5
------
Building belacoder:

    cd
    git clone https://github.com/BELABOX/belacoder.git
    cd belacoder
    make
    
Step 6
------
Building srtla:

    cd
    git clone https://github.com/BELABOX/srtla.git
    cd srtla
    make

Step 7
------
Setting up belaUI:

    cd
    git clone https://github.com/BELABOX/belaUI.git
    cd belaUI

Edit `setup.json` with the paths to the `belacoder` and `srtla` directories.

You can start the web interface to test it with:

    ruby belaUI.rb -o 0.0.0.0 -p 8080

At this point BELABOX is ready to use, assuming that you have a capture card / other v4l2 input connected. Open `http://address_of_the_jetson:8080` in a web browser. See the [belacoder readme](https://github.com/BELABOX/belacoder) for information about the available pipelines you can select in the *Encoder settings* menu of the web interface. Configure the *srtla settings* with the data for your ingest configured at *Step -1*.

After setting up and confirming that everything is working correctly, you can install belaUI as a system service that starts automatically at boot by running:

    sudo ./install_service.sh


Step 8
------

For practical use, you should configure belaUI to be automatically started at boot and use a phone to control it. Depending on your modem setup, you could make belaUI accessible either through a modem that has both USB (for the Jetson) and WiFI (for the phone) interfaces or by setting up a Wifi access point on the Jetson Nano - outside the scope of this tutorial.

If you're not confident following any of the instructions, please wait until we're able to distribute BELABOX in a more convenient format.

Once you're set up, check out [the bitrate guide](https://github.com/BELABOX/tutorial/blob/main/bitrate_guide.md).
