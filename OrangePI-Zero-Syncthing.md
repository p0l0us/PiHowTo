# Use Etcher (or similar tool) and burn to SD card.
- OS Images
  - OrangePI Zero 2: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-Zero-2.html
  - OrangePI Zero 3: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-3.html
- OrangePI Zero 2: Use Debian server image in order to support 256Gb and bigger Micro SSD Cards.
- OrangePI Zero 3: Use Ubuntu server image.
- Extract image and use Etcher or similar tool to create Micro SSD card.
- _Note: Etcher may require admin rights._

# Setup network
- Insert ssd card, connect to monitor via hdmi and power by usb-c
- _Note: Default root password is `orangepi`_
- run `sudo nmtui` and connect the device to the wifi (or use wired)
- `ifconfig` to see current devices ip address.

_From this moment you don't need monitor anymore since you can connect to the device via SSH_

# Install wireguard:
# Install syncthing
