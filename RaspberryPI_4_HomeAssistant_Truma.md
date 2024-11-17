# RaspberryPI OS installation
- Install standard RaspberryOS 32bit using the RaspberryPI Imager
- Re-enter the ssd card into reader and edit config.txt file according to Waveshare instructions https://www.waveshare.com/wiki/7HP-CAPQLED
  Basically, add following lines to the end `[All]` section
```
hdmi_force_hotplug=1 
config_hdmi_boost=10
hdmi_group=2 
hdmi_mode=87 
hdmi_cvt 1024 600 60 6 0 0 0
```
- Installed wireguard and syncthing according to OrangePI guide https://github.com/p0l0us/PiHowTo/blob/main/OrangePI-Zero-Syncthing.md
- `apt update && apt upgrade`
- Truma CP plus version is 03.00.01

# Installing home assistant (supervised to RaspberryPI OS)
- follow https://github.com/home-assistant/supervised-installer
```
apt install apparmor cifs-utils curl dbus jq libglib2.0-bin lsb-release network-manager nfs-common systemd-journal-remote systemd-resolved udisks2 wget -y
curl -fsSL get.docker.com | sh

# 32Bit
# wget -O os-agent.deb https://github.com/home-assistant/os-agent/releases/download/1.6.0/os-agent_1.6.0_linux_armv7.deb

# 64bit
wget -O os-agent.deb https://github.com/home-assistant/os-agent/releases/download/1.6.0/os-agent_1.6.0_linux_aarch64.deb

dpkg -i os-agent.deb

wget -O homeassistant-supervised.deb https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb

# in case following line doesn't work, try: BYPASS_OS_CHECK=true apt install ./homeassistant-supervised.deb
apt install ./homeassistant-supervised.deb
```
- Enable waveshare display backlight and long touch: https://www.waveshare.com/wiki/7HP-CAPQLED

# Truma LIN and MQTT
- STAUS: I'm able to connect via MQTT broker and control the temperature. \
  But I didn't connect MQTT to Home assistant.
- Connect hardware as shown here: https://github.com/danielfett/inetbox.py/tree/master
- Ensure your user is in dialout group (`sudo adduser [USERNAME] dialout`)
- Install Mosquitto MQTT service `sudo apt install mosquitto mosquitto-clients -y`
  (https://www.arubacloud.com/tutorial/how-to-install-and-secure-mosquitto-on-ubuntu-20-04.aspx#Introduction)
- Enable UART by adding lines to `/boot/firmware/config.txt`
```
enable_uart=1
dtoverlay=uart1
dtoverlay=uart3
```
- Raspberry PI 4 UART pinout:
```
        TXD RXD CTS RTS     Board Pins
uart0   14  15              8   10
uart1   14  15              8   10
uart2   0   1   2   3       27  28  (I2C)
uart3   4   5   6   7       7   29
uart4   8   9   10  11      24  21  (SPI0)
uart5   12  13  14  15      32  33  (gpio-fan)
```
- Install pip and pipx `apt install python3-pip`
- Add python bins to the path
```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
. ~/.bashrc
```
- Install intebox python tool `pip3 install inetbox_py[truma_service] --break-system-packages`
  _Note: `--break-system-packages` is required to avoid `error: externally-managed-environment`_
- Create `/etc/migro.yml`
```
broker:
  host: localhost
  port: 1883
  keepalive: 60
  
log_level: INFO

services: {}
```
- Enable truma service
```
truma_service --install
systemctl enable miqro_truma
systemctl start miqro_truma
```
- Test MQTT
```
mosquitto_pub -t 'service/truma/set/target_temp_room' -m '18'; mosquitto_pub -t 'service/truma/set/heating_mode' -m 'eco'
```
# EspHome approach
- STATUS: It works !
  The main trick was resetting the CP panel and wait looong until it pairs
- Project home: https://github.com/Fabian-Schmidt/esphome-truma_inetbox
## Install Home assistant
- Install Home Assistant to raspberrypi using the RaspberryPI Imager
- Connect to HDMI monitor and insert the flash card
- Once home assistant booted type  `login` (no password required)
- Connect to wifi using nmcli
```
nmcli radio
nmcli device wifi rescan
nmcli device wifi
nmcli device wifi connect "[YOUR_SSID]" password "[YOUR_WIFI_PASSWORD]"
# "Device 'wlan0' successfully activated with...."
nmcli con show
# To delete a connection:
# nmcli connection delete CONNECTION_NAME 
ip addr show
```
- Use wlan0 ip address to connect from web browser to `http://IP_ADDRESS:8123`
## Esp Home plugin
- Install `ESP Home` plugin to the home assistant
- Enable  `ESP Home` and add to side bar.
- Switch to  `ESP Home` plugin
## Add new Esp Home device
- Add new device, don't install yet
- Check secrets if contains correct 2G wifi SSID  and password
```
# Your Wi-Fi SSID and password
wifi_ssid: "[YOUR_SSID]"
wifi_password: "[YOUR_WIFI_PASSWORD]"
```
- Edit the ESP Home device
- Rememnber encryption key
```
api:
  encryption:
    key: "[HERE IS THE KEY]"
```
- Paste to the Esp Home config editor one of following examples: https://github.com/Fabian-Schmidt/esphome-truma_inetbox/tree/main/examples
  
<details>
  <summary>My config template:</summary>
  
```
esphome:
  name: "truma-combi"
  friendly_name: "Truma Combi 6L"

external_components:
  - source: github://Fabian-Schmidt/esphome-truma_inetbox

esp32:
  board: esp32dev

logger:
  #level: VERBOSE
  #level: DEBUG
  level: INFO

api:
  encryption:
    key: "[Paste encryption key here]"

ota:
  password: "[OTA password here]"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Truma Hotspot"
    password: "!secret wifi_password"
  
uart:
  - id: lin_uart_bus
    tx_pin: 17
    rx_pin: 16
    baud_rate: 9600
    data_bits: 8
    parity: NONE
    stop_bits: 2


climate:
  - platform: truma_inetbox
    name: "Room"
    type: ROOM
  - platform: truma_inetbox
    name: "Water"
    type: WATER

binary_sensor:
  - platform: truma_inetbox
    name: "CP Plus alive"
    type: CP_PLUS_CONNECTED
  - platform: truma_inetbox
    name: "Heater has error"
    type: HEATER_HAS_ERROR

sensor:
  - platform: truma_inetbox
    name: "Current Room Temperature"
    type: CURRENT_ROOM_TEMPERATURE
  - platform: truma_inetbox
    name: "Current Water Temperature"
    type: CURRENT_WATER_TEMPERATURE

select:
  - platform: truma_inetbox
    name: "Fan Mode"
    type: HEATER_FAN_MODE_COMBI

truma_inetbox:
  uart_id: lin_uart_bus

web_server:
  port: 80
  local: true
  version: 2
  include_internal: true
```

</details>

- Change the name
```
esphome:
  name: "truma_combi"
  friendly_name: "Truma Combi 6L"
```
- Replace encryption key by original
- Add `web_server` settings
```
web_server:
  port: 80
  local: true
  version: 2
  include_internal: true
```
- Ensure the board is correct by finding it the list https://docs.platformio.org/en/latest/boards/index.html
  or just use a generic one:
```
esp32:
  board: esp32dev
  framework:
    type: arduino
```
- More possible configuration options: https://github.com/Fabian-Schmidt/esphome-truma_inetbox/blob/main/truma.yaml
- Save and run install
- Once it is built, download the `bin` file in new format
## Install to the device
- Connect LIN-UART according to this: https://github.com/mc0110/inetbox2mqtt/blob/main/doc/ELECTRIC.md
  Note: on my Esp32 pins 16 & 17
- Connect Esp32 to the computer and determinate which serial port is used (eg. device manager on windows)
- Open https://web.esphome.io/ in Explorer or Chrome
- Press connect and select the serial port with the device
- _NOTE: You can play with "Prepare for first use" to ensure that ESP works as expected and is able to connect to your wifi_
- Choose install and select the downloaded `bin` file
- Press install
  _NOTE: Keep in mind that some ESP board requires pressing and holding the boot button until it starts erasing and installing the firmware_
## Debug and troubleshoot

# Raspberry PI 4 UART settings
```
Name:   uart0
Info:   Change the pin usage of uart0
Load:   dtoverlay=uart0,<param>=<val>
Params: txd0_pin                GPIO pin for TXD0 (14, 32 or 36 - default 14)

        rxd0_pin                GPIO pin for RXD0 (15, 33 or 37 - default 15)

        pin_func                Alternative pin function - 4(Alt0) for 14&15,
                                7(Alt3) for 32&33, 6(Alt2) for 36&37


Name:   uart1
Info:   Change the pin usage of uart1
Load:   dtoverlay=uart1,<param>=<val>
Params: txd1_pin                GPIO pin for TXD1 (14, 32 or 40 - default 14)

        rxd1_pin                GPIO pin for RXD1 (15, 33 or 41 - default 15)


Name:   uart2
Info:   Enable uart 2 on GPIOs 0-3
Load:   dtoverlay=uart2,<param>
Params: ctsrts                  Enable CTS/RTS on GPIOs 2-3 (default off)


Name:   uart3
Info:   Enable uart 3 on GPIOs 4-7
Load:   dtoverlay=uart3,<param>
Params: ctsrts                  Enable CTS/RTS on GPIOs 6-7 (default off)


Name:   uart4
Info:   Enable uart 4 on GPIOs 8-11
Load:   dtoverlay=uart4,<param>
Params: ctsrts                  Enable CTS/RTS on GPIOs 10-11 (default off)


Name:   uart5
Info:   Enable uart 5 on GPIOs 12-15
Load:   dtoverlay=uart5,<param>
Params: ctsrts                  Enable CTS/RTS on GPIOs 14-15 (default off)
```
