#### R: original article ( 16 Nov 2018 ): https://medium.com/@decrocksam/building-wpe-webkit-for-raspberry-pi-3-cdbd7b5cb362

- Pi3 B+ & Pi3 A+ are mandatory to not get lag ( additionally to optimized html/js/css & cie code )

## Building
- no binary available, can't build on RPi itself
- use build system like Buildroot or Yocto
- build systems cross-compile a complete Linux OS filled with chosen apps & tools
- instead of downloading the Raspbian image, we build our own
- Igalia team offers repos for both to easily build our own sys with WPE Webkit included
  - Buildroot: https://github.com/WebPlatformForEmbedded/buildroot
  - Yocto: https://github.com/WebPlatformForEmbedded/meta-wpe
### Building on Digital Ocean
- building an OS takes lot of time & resources
- we can spawn a virtual server as Debian server with optim CPU + 8Gb ram
- the above is $0.119/hour droplet 'll cost half a dollar if ran for 4 hours
#### COMMANDS
- we need tools:
  - apt-get update
  - apt-get install git build-essential unzip bc mtools libncurses-dev
  - dosfstools -y
- we need the Buildroot fork containing WPE Webkit:
  - git clone https://github.com/WebPlatformForEmbedded/buildroot.git
  - cd buildroot
- to build with Buildroot, we need to select a conf from configs/ & customise it if needed:
  - make raspberrypi3_wpe_defconfig # Pi3 conf with WPE included
- previously, the build process would result in a RPi img file
- due to recent changes, we only get the filesys & we need to create that image ourselves
- luckily, Github user T94T has hints on how to re-enable the creation of the image file ( https://github.com/WebPlatformForEmbedded/WPEWebKit/issues/500#issuecomment-435590466 ):
- we start by opening the customisation menu:
  - make menuconfig
- make sure the options are as below:
  ( 
    original file: https://gist.github.com/SamDecrock/283b896e3bcda97e539862f4b28a396b#file-wpe-webkit-on-pi3-csv,
    csv->md converter: https://donatstudios.com/CsvToMarkdownTable
  )

  | Operation | Location          |                                                 |                  | value | 
  |-----------|-------------------|-------------------------------------------------|------------------|-------| 
  | ENABLE    | Target packages   | Filesystem and flash utilities                  | dosfstools       |       | 
  | ENABLE    | Target packages   | Filesystem and flash utilities                  | mtools           |       | 
  | ENABLE    | Filesystem images | ext2/3/4 root filesystem                        |                  |       | 
  | SET VALUE | Filesystem images | ext2/3/4 root filesystem                        | ext2/3/4 variant | ext4  | 
  | DISABLE   | Filesystem images | initial RAM filesystem linked into linux kernel |                  |       | 
  | ENABLE    | Host utilities    | genimage                                        |                  |       | 

- edit the RPi post script:
  - nano board/raspberrypi/post-image.sh
- make sure GENIMAGE_CFG is as follows:
  - GENIMAGE_CFG="${BOARD_DIR}/genimage-${BOARD_NAME}.cfg"
- then run the build process ( may take ~3 hours on said-sapecs machine )
  - make
- if all went fine, we should have a "sdcard.img" in output/images:
  - ls output/images/ -ahl
- download this fresh image using any SFTP client
- don't forget the destroy the Digital Ocean dropled if not needed anymore
- write the image to an SD card using Etcher or some other tool


## Playing with WPE Webkit
### Moment of truth
- put SD card in RPi & boot it
- it should display ( or not ) boot logs & then a pretty simple UI with fading square saying "Acquiring network (eth0)"
- it may then say "Something went wrong (<deviceId>)" or 'interface acquired' plus an ip address
### Quick tests & debug
- there's a whole framework to control the WPE browser remotely:
  - navigating the RPi's ip address 'll display a nice little UI ( selectable URL, perfs, cpu, memory )
- to ssh into the RPi, the login is root/root


## Next steps
- custom Node.js app with fancy CSS3 UIs means configuring Buildroot to include Node.js
- having anyone ( or me ? ) building a WPE Webkit binary for Raspbian to ease prototyping
