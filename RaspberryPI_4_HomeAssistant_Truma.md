# RaspberryPI OS installation
- standard way for RPI using the RaspberryPI Imager
- Installed wireguard and syncthing according to OrangePI guide https://github.com/p0l0us/PiHowTo/blob/main/OrangePI-Zero-Syncthing.md
- `apt update && apt upgrade`
# Truma LIN and MQTT
- Note: it didn't work for me. The most likely I need truma inetbox. Original idea was that it will communicate with CP plus inet ready, but...
- Connect hardware as shown here: https://github.com/danielfett/inetbox.py/tree/master
- Ensure your user is in dialout group (`sudo adduser [USERNAME] dialout`)
- Install Mosquitto MQTT service `sudo apt install mosquitto mosquitto-clients -y`
  (https://www.arubacloud.com/tutorial/how-to-install-and-secure-mosquitto-on-ubuntu-20-04.aspx#Introduction)
- Enable UART by adding `enable_uart=1` to `/boot/firmware/config.txt`
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
# EspHome approach
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
