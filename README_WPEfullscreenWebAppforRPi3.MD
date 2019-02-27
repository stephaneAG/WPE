R: original article ( 13 Jan 2019 ): https://medium.com/@decrocksam/building-web-applications-for-wpe-webkit-using-node-js-3347146013f3
Pi3 B+ & Pi3 A+ are mandatory to not get lag ( additionally to optimized html/js/css & cie code )

## Next steps
- the build process produces the bare minimum to run an app
- quite handy when building a stable production image
- however, for dev/debug with WPEWebkit, we may need dev tools
- we'll see how to include all packages needed for dev in a Buildroot image
- among other things, to have bash, nano, screen, git, node.js, wifi, Bluetooth, serial port & cie
- IT'S IMPORTANT TO UNDERSTAND EACH STEP to be able to bake custom Buildroot images in the future

### Getting Buildroot & preparing image
- see https://github.com/stephaneAG/WPE/blob/master/README_buildingWPEWebkitforRPi3.MD
- stop RIGHT BEFORE 'then run the build process' ( make ), for further modding
- then, enable the following options: ( original file: https://gist.github.com/SamDecrock/48782fd365ee6faf32c1535d43009c04#file-wpe-webkit-buildroot-tools-csv, csv->md converter: https://donatstudios.com/CsvToMarkdownTable )

| Location             |                                                 |                             |                          |                          | 
|----------------------|-------------------------------------------------|-----------------------------|--------------------------|--------------------------| 
| System configuration | (wpe-raspberrypi) System hostame                |                             |                          | (change the hostname)    | 
| System configuration | /bin/sh (bash)                                  |                             |                          | (use bash instead of sh) | 
| Kernel               | Linux Kernel Tools                              | gpio                        |                          |                          | 
| Target packages      | Show packages that are also provided by busybox |                             |                          |                          | 
| Target packages      | Audio and video applications                    | bluez-alsa                  |                          |                          | 
| Target packages      | Development tools                               | git                         |                          |                          | 
| Target packages      | Development tools                               | git-crypt                   |                          |                          | 
| Target packages      | Development tools                               | grep                        |                          |                          | 
| Target packages      | Development tools                               | make                        |                          |                          | 
| Target packages      | Filesystem and flash utilities                  | dosfstools                  |                          |                          | 
| Target packages      | Filesystem and flash utilities                  | dosfstools                  | fatlabel                 |                          | 
| Target packages      | Filesystem and flash utilities                  | dosfstools                  | fsck.fat                 |                          | 
| Target packages      | Filesystem and flash utilities                  | dosfstools                  | mkfs.fat                 |                          | 
| Target packages      | Filesystem and flash utilities                  | e2fsprogs (accept defaults) |                          |                          | 
| Target packages      | Filesystem and flash utilities                  | e2fsprogs                   | resize2fs                |                          | 
| Target packages      | Hardware handling                               | Firmware                    | rpi-bt-firmware          |                          | 
| Target packages      | Hardware handling                               | eudev                       | enable hwdb installation |                          | 
| Target packages      | Hardware handling                               | parted                      |                          |                          | 
| Target packages      | Hardware handling                               | rfkill                      |                          |                          | 
| Target packages      | Interpreter languages and scripting             | gyp                         |                          |                          | 
| Target packages      | Interpreter languages and scripting             | nodejs                      |                          |                          | 
| Target packages      | Interpreter languages and scripting             | nodejs                      | NPM for the target       |                          | 
| Target packages      | Libraries                                       | Crypto                      | CA Certificates          |                          | 
| Target packages      | Libraries                                       | Hardware handling           | libusb                   |                          | 
| Target packages      | Libraries                                       | Networking                  | libcurl                  |                          | 
| Target packages      | Libraries                                       | Networking                  | libcurl                  | curl binary              | 
| Target packages      | Libraries                                       | Networking                  | libcurl                  | enable verbose strings   | 
| Target packages      | Libraries                                       | Other                       | libcap                   |                          | 
| Target packages      | Networking applications                         | autossh                     |                          |                          | 
| Target packages      | Networking applications                         | bluez-utils 5.x             |                          |                          | 
| Target packages      | Networking applications                         | dhcpcd                      |                          |                          | 
| Target packages      | Networking applications                         | dnsmasq                     |                          |                          | 
| Target packages      | Networking applications                         | hostapd                     |                          |                          | 
| Target packages      | Networking applications                         | hostapd                     | Enable ACS               |                          | 
| Target packages      | Networking applications                         | hostapd                     | Enable EAP               |                          | 
| Target packages      | Networking applications                         | hostapd                     | Enable WPS               |                          | 
| Target packages      | Networking applications                         | ifupdown                    |                          |                          | 
| Target packages      | Shell and utilities                             | bash                        |                          |                          | 
| Target packages      | Shell and utilities                             | screen                      |                          |                          | 
| Target packages      | Shell and utilities                             | sudo                        |                          |                          | 
| Target packages      | System tools                                    | htop                        |                          |                          | 
| Target packages      | System tools                                    | util-linux                  | mount/umount             |                          | 
| Target packages      | Text editors and viewers                        | nano                        |                          |                          | 

- all the above 'll come in handy later ;)
- R: for git over ssl to work, we need: git, curl and the CA Certificates!

### Native Node.js libraries
- ex: the original author needs 'Noble' ( Node.js to talk to BLE devices )
- 'Noble' uses the 'usb' & ' bluetooth-hci-socket' native modules
- hence, those had to be built during the Buildroot build process
- to do so, we go to 'Target packages' > 'Interpreter languages and scripting' > 'nodejs' & select 'Additional modules'
- then, we fill in the native modules we wish to use, ex:
  - usb bluetooth-hci-socket serialport node-hid
- if some modules depends on natives libraries ( ex: usb module depends on the native eudev lib ), we fill those in 'Additional module dependencies' 
  - eudev
- MAKE SURE TO SAVE BEFORE EXITING THE MENU


### EPERM Exception
- if 'no permission to build the module' problem arise: ( ex: building the bluetooth-hci-socket module )
  - fix it by editing 'buildroot/package/nodejs/nodejs.mk'
  - in it, change "$(NPM) install -g $(NODEJS_MODULES_LIST)" to "$(NPM) install -g --unsafe-perm $(NODEJS_MODULES_LIST)"
  - in other words, add the "--unsafe-perm" flag/arg
  
### Raspbian style bash prompt
- to enable the colorful bash prompt we get into logging into Raspbiana:
  - add a .bashrc & a .profile to the root
  - to make sure it's added to our Buildroot image on compilation:
    - go to: cd buildroot/system/skeleton
    - create a .bashrc with following content:
    ```bash
    export TERM=xterm-xfree86
    PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \$\[\033[00m\] '
    export NODE_PATH=/usr/lib/node_modules # to have native node.js modules loaded when a node.js app starts
    ```
    - create a .profile file that loads that .bashrc file:
    ```bash
    if [ -n "$BASH_VERSION" ]; then
      # include .bashrc if it exists
      if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
      fi
    fi
    ```


### Building the image
- run 'make' while in the /buildroot dir
- if all went fine, we should have a "sdcard.img" in output/images:
  - ls output/images/ -ahl
- download this fresh image using any SFTP client
- don't forget the destroy the Digital Ocean dropled if not needed anymore
- write the image to an SD card using Etcher or some other tool


### Running the image
- put in a Pi3 & boot
- test apparition of the "We're ready" screen
- test network connectivity


### Expanding the filesystem
- start by login over ssh using root/root:
  - ssh root@<ip_addr>
- since this is no Raspbian, there's no raspi-config to expand the filesys
- luckily, we included all the needed tools for this when configuring our Buildroot image
- run:
  - sudo fdisk /dev/mmcblk0
  - type 'p' to list partitions
  <img width="500" src="https://cdn-images-1.medium.com/max/1600/1*Z2H0_3A_cqvff7eqV-aNiA.png">
- we're gonna delete the 2nd partition & recreate it while making it span over the whole SD card
  - type 'd' to delete a partition
  - type '2' to actually delete the 2nd one
  - type 'n' to create a new partition
  - type 'p' to make it a primary partition
  - type '2' to set its partition number to 2
  - enter the 'start' sector highlighted in blue in the above image
  - accept the end sector as is
  - type 'N' to leave the signature as is
  - type 'w' to confirm ( the tool will tell us if something went wrong and didn't get applied )
- now reboot ( /!\ THIS MADE MY SYS CRASH MORE THAN ONCE .. ):
  - type 'reboot'
- to complete the fs expansion, ssh back to the Pi & run:
  - resize2fs /dev/mmcblk0p2
- after few seconds, the command should complete & we can then check our disk space:
  - df -h
  <img width="500" src="https://cdn-images-1.medium.com/max/1600/1*XKT2NMNTnVxggcUcVS_9cw.png">
- we should now have more space to work with !


### Running a Node.js app
- WPE Framework provides us a web interface that can be used to show urls in the WPE browser
- we can also use its API to make it show our own url
- to do so, we have to send a POST to /Service/WebKitBrowser/URL with the url we want to show:
  ```
  {
    "url": "https://arodic.github.io/p/jellyfish"
  }
  ```
-to get a quick working test, we have a Node.js Express app that shows the serial ports, bluetooth devices,
cpu temperature, os infos, the Bick Buck Bunnie movie & a CSS3 animation: https://github.com/SamDecrock/wpedemo.git
- using the APi described above, it changes our browser to its own url
- to run it, ssh to the Pi
- create a dir named /apps from the root dir ( we'll store our Node.js stuff in it ):
  - mkdir /apps
  - cd /apps
- then, we clone the demo app into it:
  - git clone https://github.com/SamDecrock/wpedemo.git
- then, we go into /apps/wpedemo & install its dependencies:
  - cd /apps/wpedemo
  - npm install --production
  - /!\ dont worry seeing node-gyp errors: node-gyp is trying to install the native libraries already installed globally
- to run the wbe app:
  - DEBUG=wpedemo node app.js
  
  
### Running the Node.js app on boot
- to do so, we need a startup script
- since Buildroot still uses init.d, we create a new file in /etc/init.d/:
  - nano /etc/init.d/S95wpedemo
  - the 'S95' part makee sure it's run after the WPE framework init script
  - content of the file should ressemble the following:
  ```bash
  #!/bin/sh
  export NODE_PATH=/usr/lib/node_modules # set the env var telling node where to find the globally installed node.js modules
  nohup /bin/node /apps/wpedemo/app.js > /apps/wpedemo.log 2>&1
  ```
- then, we have to make the script executable:
  - chmod +x /etc/init.d/S95wpedemo
- now, reboot the pi with 'reboot' to see our web app boot up !
  
### Complete configuration
- fork of WEP buildroot: https://github.com/SamDecrock/wpe-buildroot

