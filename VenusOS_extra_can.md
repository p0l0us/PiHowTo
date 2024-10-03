I added two more extra USB can adapters (canable and some other from aliexpress). This is how I named it to recognise them.


* Create the udev config to `/data/etc/extra-can/etc/udev/rules.d/extra-can.rules`. Modify following example up to your needs.
  I use `ID_SERIAL_SHORT` to distinguish between adapters, there are multiple solutions for sure.
```
ACTION=="add", SUBSYSTEM=="net", ENV{ID_VENDOR_ID}=="1d50", ENV{ID_MODEL_ID}=="606f", ENV{ID_SERIAL_SHORT}=="002C001E4E56511820373634", NAME="bmscan2", ENV{VE_NAME}="BMS-Can port 2"
ACTION=="add", SUBSYSTEM=="net", ENV{ID_VENDOR_ID}=="1d50", ENV{ID_MODEL_ID}=="606f", ENV{ID_SERIAL_SHORT}=="002F002E4146570420313738", NAME="n2kcan", ENV{VE_NAME}="N2K-Can port"
```

* Create file like `/data/etc/extra-can/install.sh`
```
#!/bin/bash

# CerboGX had problems to detect Multiplus without the delay.
sleep 10
cp /data/etc/extra-can/etc/udev/rules.d/extra-can.rules /etc/udev/rules.d/extra-can.rules
udevadm control --reload-rules && udevadm trigger
```
* Execute the file from `/data/rcS.local`
```
#!/bin/bash

nohup /data/etc/extra-can/install.sh > /dev/null &
```
* Make sure `/data/rcS.local` is executable
```
chmod 755 /data/rcS.local
```
