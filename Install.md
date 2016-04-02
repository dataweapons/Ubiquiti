sudo apt-get install oracle-java8-jdk

Turning a Raspberry Pi into a UniFi controller appliance (UniFi 4, Raspbian Jessie, Oracle Java 8) en

By ERIKvanPAASSEN on Tuesday 29 December 2015 22:09 - Comments (16)
Category: UniFi, Views: 4.033

In March 2014, I wrote a blog post about running the controller software for Ubiquiti UniFi access points on a Raspberry Pi. This was just after the first hard-float version of the Oracle Java 7 JDK for Raspbian was released and running the UniFi controller on Linux was not officially supported yet. The blog posts was one of the very first available on this topic. I was surprised by the amount of reactions I got, as I considered the setup to be just a proof of concept.

However, after months of service, my Raspberry Pi still runs stable. But over the time, a lot of things have changed. The original tutorial used version 2 of the controller software, while the most recent version available today is 4. With Raspbian Jessie, MongoDB became available from the repositories. The Oracle Java 8 JDK has been released. And so on. After all, I decided it was time to rewrite my tutorial and here it is.

http://static.tweakers.net/ext/f/KON6T2YwLiY7CSKN4sJmjkN4/full.png
Screenshot of the UniFi controller web interface.


I'm not providing ready-to-go images as I think it's good to know how everything is set up when problems might arise. I'll try to explain every step carefully, but just copy-pasting all commands should already be enough to yield a working controller though.


Prerequisites
Raspberry Pi with at least 512 MB RAM. For example: RPi 1 model B rev. 2, RPi 1 model B+ or RPi 2 model B.
A compatible SD card. I'd recommend using a large card (16 GB) to avoid excessive write wear on it.

Necessary steps
We'll need to take the following steps in order to get your UniFi controller up and running.
Setup and configure the Raspbian OS
Install the dependencies
Install the UniFi controller
Modify the start script in order to use Oracle Java 8 instead of OpenJDK 7
Configure the controller to run as its own user instead of as root
Running the controller


Raspbian setup
The first step in the process will be getting your Pi up and running with Raspbian Jessie. You can download an image on the Raspbian downloads page. I'll be using the Lite image, a minimal image wihout graphical user interface, as that's just what we need for a headless device. If you don't know how to write the image to your SD card, I'd recommend you to take a look at the wiki page on this topic. After you've flashed Raspbian to the card, it's time to boot up your Pi and log in with username pi and password raspberry. If you like, you can also log in through SSH, which is enabled out of the box.

After the image has been flashed, we still need to expand the filesystem to the full size of the SD card to be able to use all the space provided by the card. To do so, run:
sudo raspi-config

Choose Expand Filesystem to adjust the filesystem size. While we're in this neat little configuration tool, you might also want to adjust the amount of RAM assigned to the graphics adapter. You can do so by choosing Advanced Options > Memory split. As we're running headless, 16MB would be more than enough. Reboot the device for the changes to take effect.

You may also want to configure the timezone you're living in, using:
sudo dpkg-reconfigure .
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install oracle-java8-jdk

echo 'deb http://www.ubnt.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/unifi.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv C0A52C50
sudo apt-get update
sudo apt-get install unifi


sudo sed -i 's@^set_java_home$@#set_java_home\n\n# Use Oracle Java 8 JVM instead.\nJAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm-vfp-hflt@' /usr/lib/unifi/bin/unifi.init

sudo cp /lib/systemd/system/unifi.service /etc/systemd/system/
sudo sed -i '/^\[Service\]$/a Environment=JAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm-vfp-hflt' /etc/systemd/system/unifi.service

sudo sed -i 's@-Xmx1024M@-Xmx384M@' /usr/lib/unifi/bin/unifi.init
sudo sed -i 's@-Xmx1024M@-Xmx768M@' /usr/lib/unifi/bin/unifi.init
sudo useradd -r unifi


sudo chown -R unifi:unifi /var/lib/unifi /var/log/unifi /var/run/unifi /usr/lib/unifi/work
sudo sed -i '/^\[Service\]$/a User=unifi' /etc/systemd/system/unifi.service
sudo echo 'ENABLE_MONGODB=no' | sudo tee -a /etc/mongodb.conf > /dev/null

