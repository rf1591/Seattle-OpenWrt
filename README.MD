# Seattle on OpenWrt

## Overview 
Seattle is an open research and educational testbed that utilizes computational resources provided by end users on their existing devices. This project walks through an installation of OpenWrt on a TP-Link TL-WDR3600 router, the installation of a custom-built Seattle package, and the results of some experiments run using the Seattle testbed on a router. This project is intended to work for OpenWrt Chaos Calmer release (v. 15.05.1). Change the download links and directory names appropriately for future releases. 

## Installing OpenWrt
Install the latest release of OpenWrt on the router. 
-  Documentation can be found [here](https://openwrt.org/)
-  OpenWrt releases and downloads for an SDK, custom ImageBuilders, and installation guides for other devices can be found [here](https://downloads.openwrt.org/)

##### Pre-built Image
OpenWrt provides documentation for installing on the WDR3600 router on their website: https://wiki.openwrt.org/toh/tp-link/tl-wdr4300. This installs a pre-built image on a device with factory firmware. 

##### Custom Image
Customize a light-weight image using OpenWrt's custom ImageBuilder.

* Download ImageBuilder 

	```
	cd ~
	mkdir openwrt && cd openwrt
	wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/OpenWrt-ImageBuilder-15.05.1-ar71xx-generic.Linux-x86_64.tar.bz2
	tar -xvjf OpenWrt-ImageBuilder-15.05.1-ar71xx-generic.Linux-x86_64.tar.bz2
	cd OpenWrt-ImageBuilder-15.05.1-ar71xx-generic.Linux-x86_64
	```

* Make images

	```
	sudo make image PROFILE=TLWDR4300 PACKAGES="-ppp zlib libffi libpthread python-light python-compiler python-codecs python-ctypes python-logging python-openssl python-xml"
	```

* Copy images to router

	```
	cd bin/ar71xx
	scp openwrt-15.05.1.ar71xx-generic-tl-wdr3600-v1-squashfs-factory.bin root@192.168.1.1:/tmp
	```

* Install the firmware (ssh into router first)

	```
	mtd -r write /tmp/openwrt-15.05.1.ar71xx-generic-tl-wdr3600-v1-squashfs-factory.bin firmware
	```

##### Flash Upgrade 

If a previous version of OpenWrt exists on the router, ssh into the device and rewrite

```
cd /tmp 
wget downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/openwrt-15.05.1-ar71xx-generic-tl-wdr3600-v1-squashfs-factory.bin
mtd -r write /tmp/openwrt-15.05.1-ar71xx-generic-tl-wdr3600-v1-squashfs-factory.bin firmware
```



## Installing Seattle

### Building a custom package with OpenWrt's SDK

* Installing OpenWrt SDK 

	> Construct a Seattle package for OpenWrt by installing a precompiled OpenWrt toolchain used to cross compile single packages on the system. 

	```
	cd ~ 
	wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/OpenWrt-SDK-15.05.1-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2
	tar -xvf OpenWrt-SDK-15.05.1-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2
	```

* Downloading Seattle code and Makefile

	> Change into the package directory of the SDK, and clone the Seattle code. 

	```
	cd OpenWrt-SDK-15.05.1-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64/package
	git clone https://github.com/rf1591/Seattle-OpenWrt
	mv Seattle-OpenWrt seattle
	```

* Building Seattle .ipk

	> Return to the SDK directory and execute make. 

	```
	cd ..
	make clean && make world
	```

	> If make is executed error-free, the .ipk should be found in /bin/ar71xx/packages/base.

* Copy the package to the router

	> Copy the ipk to /tmp in the router. Install the package using OpenWrt's package manager `opkg`. 

	```
	scp seattle_1.0-1_ar71xx.ipk root@192.168.1.1:/tmp
	opkg install /tmp/seattle_1.0-1_ar71xx.ipk 
	```


### Custom Seattle Installer for OpenWrt

1. Download the Seattle tarball. 
2. Copy the tarball to OpenWrt with `scp seattle_openwrt.tar.gz root@192.168.1.1:/`  
3. Ssh into the router and extract the tarball with `tar -xvzf seattle_openwrt.tar.gz`
5. Install the packages required for Seattle installation. <br />
	`opkg install python-light python-compiler python-ctypes python-logging python-openssl`
6. Navigate to the directory named _seattle_ and run `./install.sh`
7. To check that Seattle is running, try running: `ps | grep nmmain.py | grep -v grep`

Seattle Dependencies
- python-light
- python-compiler
- python-codecs
- python-ctypes
- python-logging
- python-openssl

Additional Packages
- git-http (for unit-testing)


## Errors
The following are descriptions of common issues/errors encountered when trying to install OpenWrt/Seattle onto a router. Included are how they were resolved.

* Tried to execute make in OS X terminal. Resulted in heaps of "cannot execute binary file" errors. 
  - Wrong OS. Rebuild in Linux. 

* After deleting some packages on the router to free up memory, router became read-only system.
  - Remount the system read-write 
  - `$ mount -o remount,rw /`

* After flashing router with OpenWrt, system detected a "host key change"
  - Generate new host keys to authenticate user system.
  - `ssh-keygen -R 192.168.1.1`

* Abrupt exit from Seattle installation from .ipk due to insufficient memory in `/overlay`
  - Change destination of Seattle installation to /tmp 
  - `opkg install seattle_1.0-1_ar71xx.ipk -d ram`





