# Use Etcher (or similar tool) and burn to SD card.
- Instructions here: https://github.com/victronenergy/venus/wiki/raspberrypi-install-venus-image
- OS Images here: https://updates.victronenergy.com/feeds/venus/release/images/raspberrypi2/
- Download `venus-image-large-raspberrypi2.wic.gz`. Large contains Red and Signal support.
- _Note: Etcher may require admin rights._

# Boot RPI
- Insert the sd card to the RPI
- Connect USB keyboard
- Connect HDMI monitor
- Connect USB power cable
- Setup Wifi connection if ethernet not available
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
-  Follow instructions: https://community.victronenergy.com/articles/211164/howto-venus-os-setting-up-wireguard.html 

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
- Setup dbus-serialbattery to use can0. 
  - To file `/data/etc/dbus-serialbattery/config.ini` add `line CAN_PORT = can0`
  - Reinstall dbus-serial battery drivers to enable CAN venus driver: `/data/etc/dbus-serialbattery/reinstall-local.sh`
  - Confirm can drivers is listed in the command output
  - Reboot
  - See logs:  `tail -F -n 100 /data/log/dbus-canbattery.can0/current | tai64nlocal`


## USB-CAN bus adapter (not tested):
- Connect the adapter to the USB
- Install package helper script https://github.com/kwindrem/SetupHelper
- In Venus UI Navigate to Settings -> Package manager and install `VeCanSetup` package
- _Note: Use main branch if you wish the more recent, but also more unstable version_


