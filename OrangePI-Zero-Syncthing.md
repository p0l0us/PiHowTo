# Burn OS image the the SD card
- Select OS Image from
  - OrangePI Zero 2: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-Zero-2.html
  - OrangePI Zero 3: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-3.html
- OrangePI Zero 2: Use Debian server image in order to support 256Gb and bigger Micro SSD Cards.
- OrangePI Zero 3: Use Ubuntu server image.
- Extract compressed image file
- Use Balena Etcher or similar tool to create Micro SD card.
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
- make sure the hostname is also in `/etc/hosts`
```
127.0.0.1 [hostname]
```
- update the system
```
sudo apt update
sudo apt upgrade
```
- reboot, after reboot the device should automatically connect to the wifi 

_From this moment you don't need monitor anymore since you can connect to the device via SSH and your own username_

# Setup firewall (ufw)
- Install ufw `sudo apt install ufw`
- Ensure ssh is allowed `sudo ufw allow ssh`
- Enable ufw `sudo ufw enable`
- Check status `sudo ufw status`

# Create authorized ssh keys
- Generate your private & public key pair for ssh (eg. Putty-gen)
- Sign to the server ssh with your account
- Run following commands
```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
and paste your generated public key to the `authorized_keys` file.
- Make sure following is in your `/etc/ssh/sshd_config` file
```
HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes=+ssh-rsa

RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile  %h/.ssh/authorized_keys
```
- restart ssh server `/etc/init.d/ssh restart`
  
# Install wireguard:
= Read documentation for details https://www.wireguard.com/install/
- Assuming you have already wireguard server
- Install wireguard `sudo apt install wireguard`
- _Note: if you have RaspberryPI, also install resolvconf `sudo apt install resolvconf`
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
= Read documentation for details https://docs.syncthing.net/intro/getting-started.html
  or search internet for full howto.
- Install syncthing...
```
sudo apt-get update
sudo apt-get install syncthing
```
- ...Or you can use syncthing repository to get most recent version
```
sudo apt-get install apt-transport-https ca-certificates
curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
# 00654A3E should be last 8 chars from the syncthing key which you get by running command
# sudo apt-key list
sudo apt-key export 00654A3E | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/syncthing.gpg
sudo apt-get update
sudo apt-get install syncthing
```
- Set firewall rules
```
sudo ufw allow 22000/tcp
sudo ufw allow 21027/udp
sudo ufw allow 8384/tcp
sudo ufw status
```
_Note: consider changing the syncthing port `8383` to something else_
- Start and install syncthing service to run under you user
```
sudo systemctl enable syncthing@[username].service
sudo systemctl start syncthing@[username].service
```
- Check status
```
sudo systemctl status syncthing@[username].service
```
- Edit syncthing config file and set the GUI IP address
  `/home/[username]/.config/state/syncthing/config.xml`
  _Note: can be also in `.local/syncthing`_
```
...
<gui enabled="true" tls="false" debugging="false">
    <address>0.0.0.0:8384</address>
    ...
</gui>
...
```
- Restart syncthing `sudo systemctl restart syncthing@[username].service`
- Navigate your web browser to Syncthing UI at https://[some ip address]:8384
 
# Other ideas
## Comitup project
- https://github.com/davesteele/comitup
- Unstable, I didn't make it work yet
- I should give it a try later with clean installation.
- Maybe it has issues with latest bookworm debian ?
- Author is rewriting it... 

## Webmin
- https://webmin.com/
- Easy installation
- Quite slow
- Doesn't allow set wifi

## Cockpit
- https://cockpit-project.org/
- Easy installation
- Less feature then webmin
- Doesn't allow set wifi
