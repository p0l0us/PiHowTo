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
- Change your root password `sudo passwd root`
- Create your own user so you don't use default orangepi anymore.
```
sudo adduser [username]
sudo usermod -aG sudo [username]
sudo su [username]
```
- _Note: Default root password is `orangepi`_
- remove `orangepi` user `sudo deluser --remove-home orangepi`
- run `sudo nmtui`
  - connect the device to the wifi (or use wired)
  - set a meaningful device host name 
- `ifconfig` to see current devices ip address.
- update the system
```
sudo apt update
sudo apt upgrade
```
- reboot, after reboot the device should automatically connect to the wifi 

_From this moment you don't need monitor anymore since you can connect to the device via SSH and your own username_

# Install wireguard:
- Assuming you have already wireguard server
- Install wireguard `sudo apt install wireguard`
- Create private key file
  - If you don't have any, create one `wg genkey | sudo tee /etc/wireguard/private.key`
  - If you have one prepared, store it in the file `sudo nano /etc/wireguard/private.key`
- Make the private key secure `sudo chmod go= /etc/wireguard/private.key`
- To get public key for the server  `sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key`
- Edit the wireguard config file `sudo nano /etc/wireguard/wg0.conf`
```
[Interface]
PrivateKey = [Insert the client private key]
Address = [Insert client IPv4 and IPv6 addresses]
DNS = 8.8.8.8  # if needed

[Peer]
PublicKey = [Insert public key of the server]
AllowedIPs = [Insert IPv4 and IPv6 addresses to be routed through the wireguard]
Endpoint = [Insert server ip address and port]
PersistentKeepalive = 15
```
- Enable ip forwarding `sudo nano /etc/sysctl.conf`
```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```
- and run `sudo sysctl -p`
- Enable and run wireguard service
```
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```
- Test wireguard service
```
sudo systemctl status wg-quick@wg0.service
sudo ifconfig
sudo ping [server wireguard internal ip]
```
- Set custom routing rules or iptables for wireguard if needed to `wg0` using `PostUp` and `PreDown` entries

_Now you should be able to connect to your device anytime it is connected to the internet via wireguard VPN_

# Install syncthing
