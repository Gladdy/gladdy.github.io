---
layout: post
title: Setting up the Altera Quartus JTAG Programmer on Ubuntu 16.04
date:   2017-03-18 21:40:19
abstract: ""
tags: fpga hardware altera
---

## Preparation:
Run `dmesg -w` to show you the logstream of USB devices connecting and disconnecting

## Before launching Quartus
Before starting the JTAG programmer in Quartus, the `dmesg` entries should look like this:

{% highlight C %}
[27810.071135] usb 1-1.2: new high-speed USB device number 18 using ehci-pci
[27810.163900] usb 1-1.2: New USB device found, idVendor=09fb, idProduct=6810
[27810.163905] usb 1-1.2: New USB device strings: Mfr=0, Product=0, SerialNumber=0
{% endhighlight %}

## Start the Quartus Programmer
Starting the programmer will do 2 things:
- Load the JTAG server (`jtagd --user-start --config ~/.jtagd.conf`) 
- Load the driver for the USB connection

{% highlight C %}
[28122.900173] usb 1-1.2: new high-speed USB device number 25 using ehci-pci
[28123.010811] usb 1-1.2: New USB device found, idVendor=09fb, idProduct=6010
[28123.010818] usb 1-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[28123.010822] usb 1-1.2: Product: DE-SoC
[28123.010825] usb 1-1.2: Manufacturer: Altera
[28123.010828] usb 1-1.2: SerialNumber: DE-SoC-003-03503
{% endhighlight %}


## Unable to select the device in the Quartus programmer?
The most likely issue is that currently, you do not have permission to access the device, as by default, USB devices are only accessible as root. Luckily, (as long as you have root access in your system), you can fix this.

1. Figure out where the USB device lives on the filesystem. In UNIX, everything is a file, including USB devices. Go to `/dev/bus/usb`, where you will find several folders. The first level indicates the (internal) USB hub, the second level the device number. As seen before, my device has number 25, therefore there is only 1 candidate: `/dev/bus/usb/001/025`.

2. Set read-write permissions for all users on that file, eg `sudo chmod 666 /dev/bus/usb/001/025`. This has to be done every time you plug in your FPGA, which can get very tedious. 

3. Luckily, the linux kernel devs have thought of this and added a feature called udev rules. These are commands that are run, as root, when a USB device of a specific signature connects. Create the file `/etc/udev/rules.d/52-usbblaster.rules`. Any number between 0 and 100 and name will do but usually the high and low 10 are reserved for system purposes. Do make sure it adheres to the format `{number}-{name}.rules`. In this file create one udev entry. Mine is shown below. Ensure that the idVendor and idProduct actually match the codes displayed in your `dmesg` output. The rest of the entry is self-explanatory. Finally, restart the udev service by `sudo service udev restart`, forcing it to reload the udev rules.

4. Check whether your udev rules has worked. Replug your USB connection and verify that your device now has read and write permissions for all users, eg:

{% highlight bash %}
# The USB Blaster UDEV entry in /etc/udev/rules.d/52-usbblaster.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="09fb", ATTR{idProduct}=="6010", MODE="0666"
{% endhighlight %}

{% highlight bash %}
$ tree /dev/bus/usb
.
├── 001
│   ├── 001
│   ├── 002
│   ├── 003
│   ├── 005
│   ├── 013
│   └── 025
├── 002
│   ├── 001
│   └── 002
├── 003
│   └── 001
└── 004
    └── 001
{% endhighlight %}


{% highlight bash %}
# After successfully applying the udev rule or manually setting the permission
ls -lh /dev/bus/usb/001
crw-rw-r-- 1 root root 189,  0 Mar 18 21:22 001
crw-rw-r-- 1 root root 189,  1 Mar 18 21:22 002
crw-rw-r-- 1 root root 189,  2 Mar 18 21:22 003
crw-rw-r-- 1 root root 189,  4 Mar 18 21:22 005
crw-rw-r-- 1 root root 189, 12 Mar 18 21:22 013
crw-rw-rw- 1 root root 189, 35 Mar 18 21:27 036  # all users have read/write permission
{% endhighlight %}

## Any further issues?
I believe this should be a comprehensive guide for getting Altera FPGAs working on Ubuntu 16.04. If you still have issues, please leave a comment so we can resolve it and add it to the guide.
