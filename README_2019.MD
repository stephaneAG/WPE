As of 27/02/2019, I now have something working thanks to the following tutorials:

R: comparison of RPi models: http://socialcompare.com/fr/comparison/raspberrypi-models-comparison

#### Maybe still relevant in order to use a small screen, a mouse, or a touchscreen ..
- https://github.com/balena-io-projects/balena-wpe#loading-a-url

#### The studied & partially applied knowledge

- https://medium.com/@decrocksam/building-wpe-webkit-for-raspberry-pi-3-cdbd7b5cb362

For the above one, the author provides us with a 124Mo "sdcard.img" wich by default boots in ~15 seconds & 
then waits for the network to be connected in order to offer a remote ui accessible at the device's ip address.
The corresponding shorter how-to resides at: [README_buildingWPEWebkitforRPi3.MD](README_buildingWPEWebkitforRPi3.MD)

- https://medium.com/@decrocksam/building-web-applications-for-wpe-webkit-using-node-js-3347146013f3

For this one, the author provides us a 231Mo "wpewebkit-and-tools-2.img" that seems to give me troubles ( it seems "reboot" 
is not making the device reboot but crash instead ), ssh refuses connection & can't resize the partition .. but this one is supposed
to offer nodejs as well as a user other than root ;)
The corresponding shorter how-to resides at: [README_WPEfullscreenWebAppforRPi3.MD](README_WPEfullscreenWebAppforRPi3.MD) 

Transcripts are in the related readmes ;p
