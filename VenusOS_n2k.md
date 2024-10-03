Just quick notes how did I try install n2k with signalk to CerboGX / VenusOS

## Configuration
* inspired by: https://dayba.wordpress.com/2017/05/25/connecting-socket-can-adapter-to-signalk/
```
cd /data
curl -L -o canboat.zip http://github.com/canboat/canboat/zipball/master
mv canboat-canboat-9914fdb/ canboat  # the folder name will differ the most likely
cd canboat
make
make install
```
* debug this command first `candump n2kcan | candump2analyzer`, if it works, then update signalk settings.json
* location of SinglaK is /data/conf/signalk/settings.json, put following to the `pipeProviders` array. Change n2kcan to what ever you use.
```
{
  "id": "n2kcan",
  "pipeElements": [
    {
      "type": "providers/execute",
      "options": {
        "command": "candump n2kcan | candump2analyzer"
      }
    },
    {
      "type": "providers/liner",
      "options": {
        "rawlogging": true,
        "logdir": "logs",
        "discriminator": "2"
      }
    },
    {
      "type": "providers/n2kAnalyzer"
    },
    {
      "type": "providers/n2k-signalk"
    }
  ]
```

* After signalK restart you should see the n2kcan in SignalK data providers via web ui.


### notes:
* I'm using beta 1.5, but should work with 1.4 venus os as well.
* I'm using extra canbus adapter (canable USB) which I re-name using udev to n2kcan
* There is also this project, but it didn't work on first shot and I didn't spend time with debugging
https://github.com/chacal/signalk-socketcan-device
