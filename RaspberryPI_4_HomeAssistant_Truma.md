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
