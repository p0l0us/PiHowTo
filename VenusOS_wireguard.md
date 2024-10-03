Originally found here: https://community.victronenergy.com/articles/211164/howto-venus-os-setting-up-wireguard.html

1. Create install file `/data/etc/wireguard/run.sh`
(put your own IP address)
```
#!/bin/bash

ip link add dev wg0 type wireguard
ip address add dev wg0 10.0.0.2/16
wg setconf wg0 /data/etc/wireguard/wg0.conf
ip link set up dev wg0
```
2. And execute the `run.sh` file from `/data/rc.local` file.
```
#!/bin/bash

/data/etc/wireguard/run.sh
```
3. Make sure the `/data/rcS.local` file is executable
```
chmod 755 /data/rcS.local
```
4. Create installation script `/data/etc/wireguard/install.sh`
```
#!/bin/bash

# Better to delay this job
sleep 10

running_file_name=$(basename "$0")
echo "-"
echo "[Running '$running_file_name']"
 
if [ -x "$(command -v wg)" ] ; then
  echo "wireguard already installed"
  exit
fi
 
if [ ! -x "$(command -v install)" ] ; then
  echo "command install not found, checking internet connection"
  curl --output /dev/null --silent --retry 15 --retry-max-time 120 --retry-connrefused -w "\n" "8.8.8.8" #wait for internet connection
  exit_status=$?
  if [ $exit_status -ne 0 ]; then
    echo "failed detecting connection to internet"
    exit 100
  fi
  echo "installing coreutils"
  opkg update
  opkg install coreutils
fi
 
if [ -x "$(command -v install)" ] ; then 
  echo "installing wireguard"
  if [ ! -d "wireguard-tools" ] ; then
    echo "cloning wireguard git repository"
    git clone https://git.zx2c4.com/wireguard-tools
  else
    echo "wireguard git repository already exists"
  fi
  make -C wireguard-tools/src -j$(nproc)
  sudo make -C wireguard-tools/src install
  echo "wireguard installed"
else
  echo "install command not found, failed installing"
fi
```
5. And execute the file from `/data/rcS.local`
```
#!/bin/bash

nohup /data/etc/wireguard/install.sh > /dev/null &
```
6. And make sure `/data/rcS.local` is executable
```
chmod 755 /data/rcS.local
```
7. Create wireguard configuration file `/data/etc/wireguard/wg0.conf`
```
[Interface]
PrivateKey = <YOUR PRIVATE KEY HERE>

[Peer]
PublicKey = <SERVER PUBLIC KEY HERE>
AllowedIPs = <WIREGUARD NETWORK IP RANGE eg. 10.0.0.0/16>
Endpoint = <WIREGAURD SERVER IP : SERVER PORT>
PersistentKeepalive = 15
```
7. reboot
