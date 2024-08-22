# Use Etcher (or similar tool) and burn to SD card.
- Instructions here: https://github.com/victronenergy/venus/wiki/raspberrypi-install-venus-image
- OS Images here: https://updates.victronenergy.com/feeds/venus/release/images/raspberrypi2/
- Download `venus-image-large-raspberrypi2.wic.gz`. Large contains Red and Signal support.
- _Note: Etcher may require admin rights._

# Boot RPI and connect to WIFI
- Insert the sd card to the RPI
- Connect USB keyboard
- Connect HDMI monitor
- Connect USB power cable
- Run `connmanctl` and use following commands to conect to the WIFI:
```
scan wifi
agent on

# this command will list available wifi access points
services 

# fill with selected wifi_xxxx data.
# write first part and use TAB key to autofill (bash style)
connect wifi_.........

# wait approx 30s for the result (connected or fail)
# if connection fail, try different wifi router (eg. mobile phone)
```
- Show Venus UI on the HDMI display
  - Remount `/` as writable: `mount -o remount,rw /`
  - Disable headless mode: `mv /etc/venus/headless /etc/venus/headless.off`
  - Reboot
  - You should see VenusOS UI on the HDMI monitor connected to RPI
  - _Note: use arrows to navigate within the UI:_
    - _ENTER key means home_
    - _Spacebar means confirm_
    - _ESC navigates to system overlay_
    - _LEFT arrow means back_
    - _RIGHT arrow selects item_
    
# Navigate to Settings, Wi-Fi and connect to the network
- Use the device IP address in the browser to see the Victron Venus Web UI
- Enable SSH
- Navigate to Settings, General
- Keep selected “Access level” entry and hold the right arrow until the access level changes to Superuser.
- Set root password
- Enable SSH on LAN
- Reboot
- (optional) Enable headless mode
  - Remount `/` as writable: `mount -o remount,rw /`
  - Enable headless mode: `mv /etc/venus/headless.off /etc/venus/headless`
  - Reboot
    
# Install wireguard
- Follow instructions: https://community.victronenergy.com/articles/211164/howto-venus-os-setting-up-wireguard.html
- Create wireguard installation script `/data/wireguard/install.sh`:
```
#!/bin/bash

running_file_name=$(basename "$0")
echo "-"
echo "[Running '$running_file_name']"

if [ -x "$(command -v wg)" ] ; then
  echo "Wireguard already installed"
  exit
fi

if [ ! -x "$(command -v install)" ] ; then
  echo "command install not found, checking internet connection"
  curl --output /dev/null --silent --retry 15 --retry-max-time 120 --retry-connrefused -w "\n" "8.8.8.8" #wait for internet connection
  exit_status=$?
  if [ $exit_status -ne 0 ]; then
    echo "Connection to internet failed !"
    exit 100
  fi
  echo "Installing coreutils"
  opkg update
  opkg install coreutils
fi

if [ -x "$(command -v install)" ] ; then
  echo "installing wireguard"
  if [ ! -d "wireguard-tools" ] ; then
    echo "Cloning wireguard git repository"
    git clone https://git.zx2c4.com/wireguard-tools
  else
    echo "Wireguard git repository already exists"
  fi
  make -C wireguard-tools/src -j$(nproc)
  sudo make -C wireguard-tools/src install
  echo "Wireguard installed"
else
  echo "Install command not found, installation failed !"
fi
```
- And make it executable `chmod 755 /data/wireguard/install.sh`
- Add wireguard install script to `/data/rc.local` file. Mine looks like:
```
#!/bin/bash

source /data/wireguard/install.sh
source /data/wireguard/run.sh
```
- And make `/data/rc.local` executable if it's not: `chmod 755 /data/rc.local`
- Create `/data/wireguard/wg0.conf` according to your needs like:
```
[Interface]
PrivateKey = <enter client private key here>

[Peer]
PublicKey = <enter server public key here>
AllowedIPs = <server network address>/24
Endpoint = <enter your wiregaurd server pulbic IP address>
PersistentKeepalive = 15
```
- Create script `/data/wireguard/run.sh` to initialize wg0 interface with your ip address:
```
ip link add dev wg0 type wireguard
ip address add dev wg0 <client ip address>/24
wg setconf wg0 /data/wireguard/wg0.conf
ip link set up dev wg0
```
- And make the run script readable `chmod 755 /data/wireguard/run.sh`

# Install dbus-serialbattery driver
- Install the CAN module firstly (see bellow)
- Project location: https://github.com/Louisvdw/dbus-serialbattery 
- Detailed installation instructions: https://louisvdw.github.io/dbus-serialbattery/general/install
- Enable DVCC on the venus device using the Victron venus UI.
- Run following commands:
```
wget -O /tmp/install.sh https://raw.githubusercontent.com/Louisvdw/dbus-serialbattery/master/etc/dbus-serialbattery/install.sh
bash /tmp/install.sh
```
- Select Latest or Nightly main version. 
- _Note: If you meet problems with `wheel` module installation, use Nightly where it is fixed._
- Read and follow instructions on the screen
- Read also default config file: `https://github.com/Louisvdw/dbus-serialbattery/blob/master/etc/dbus-serialbattery/config.default.ini`
- Troubleshooting:
  - For troubleshooting read: `https://louisvdw.github.io/dbus-serialbattery/troubleshoot/`_
  - Add `LOGGING = DEBUG` entry to `/data/etc/dbus-serialbattery/config.ini` and reboot
  - See logs: 
    - `tail -F -n 100 /data/log/dbus-canbattery.can0/current | tai64nlocal`
- Connect to DalyBMS
  - Set can speed to 250kbps
  - Test with candump can0
  - Wire to DalyBMS

## Setup dbus-serialbattery to use can0. 
  - Make sure your CAN port works and is available. See section `Install CAN module` if you need any hint.
  - To file `/data/etc/dbus-serialbattery/config.ini` add `line CAN_PORT = can0`
  - Reinstall dbus-serial battery drivers to enable CAN venus driver: `/data/etc/dbus-serialbattery/reinstall-local.sh`
  - Confirm can drivers is listed in the command output
  - Reboot
  - See logs:  `tail -F -n 100 /data/log/dbus-canbattery.can0/current | tai64nlocal`
    
# Install CAN module
## CAN-SPI module MCP2515 with TJA1050
- Make sure your adapter with TJA1050 support 3.3V VCC provided by RPI https://forums.raspberrypi.com/viewtopic.php?t=141052
- Connect the modules to SPI inputs + 5V: https://domoticx.com/raspberry-pi-can-bus-communicatie-gpio/  
- To `/u-boot/config.txt` add MCP2515 overlay `dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25`
- _Note: Use osscillator frequency according to your device_
- Don’t forget to put a terminator jumper or add your own termination resistor.
- Alternatively (not tested):
  - Follow wiring in this video: https://www.youtube.com/watch?v=fXiOIUZtV10
  - To `/u-boot/config.txt` add MCP2515 overlay `dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=12`
- Reboot
- Connect to via SSH
- Test can0 (or other interface): 
```
candump can0
cansend can0 512#aabbcc00dd
```

## Waveshare RS485 CAN pHAT
- https://www.waveshare.com/wiki/RS485_CAN_HAT
- Connect the HAT to your rpi
- To `all` section of the `/u-boot/config.txt` add MCP2515 overlay and make sure `dtparam=spi=on` is already there.
```
[all]
dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=12000000,interrupt=25,spimaxfrequency=2000000
# or
# dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25,spimaxfrequency=1000000
```
- Test can0 (or other interface): 
```
candump can0
cansend can0 512#aabbcc00dd
```

## USB-CAN bus adapter:
- _Note: I didn't try with Raspberry PI yet. It works with CerboGX_
- Connect the adapter to the USB
- _Note: I'm using two different cheap CAN adapters with Victron CerboGX (one is Canable USB-C other is a MicroUSB). Both works without any configuration._
- When using more than one USB-CAN adapter, canX port names are assigned randomly, and VenusOS swaps it's settings. To fix this set own udev rules based on CAN adapter serial number or other unique identifier.

## USB options untested:
- Install package helper script https://github.com/kwindrem/SetupHelper
- In Venus UI Navigate to Settings -> Package manager and install `VeCanSetup` package
- _Note: Use main branch if you wish the more recent, but also more unstable version_

