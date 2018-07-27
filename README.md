# HomeKit Python [![Build Status](https://travis-ci.org/jlusiardi/homekit_python.svg?branch=master)](https://travis-ci.org/jlusiardi/homekit_python) [![Coverage Status](https://coveralls.io/repos/github/jlusiardi/homekit_python/badge.svg?branch=master)](https://coveralls.io/github/jlusiardi/homekit_python?branch=master)

With this code it is possible to implement either a HomeKit Accessory or simulate a
HomeKit Controller.

**Limitations**

 * This code only works with HomeKit IP Accessories. no Bluetooth LE Accessories (yet)!
 * The simulated Controller can handle events, but the Accessory is missing this feature

The code presented in this repository was created based on release R1 from 2017-06-07.

# Installation

Since the code relies on **gmpy2** for large numbers some development libraries and a compiler is required:

So for debian:
```bash
apt install libgmp-dev libmpfr-dev libmpc-dev libffi-dev build-essential python3-pip python3-dev
```

After that use **pip3** to install the package:

```bash
pip3 install --user homekit
```

# HomeKit Accessory
This package helps in creating a custom HomeKit Accessory.

The demonstration uses this JSON in `~/.homekit/demoserver.json`: 
```json
{
  "name": "DemoAccessory",
  "host_ip": "$YOUR IP",
  "host_port": 8080,
  "accessory_pairing_id": "12:00:00:00:00:00",
  "accessory_pin": "031-45-154",
  "peers": {},
  "unsuccessful_tries": 0,
  "c#": 0,
  "category": "Lightbulb"

}
```

Now let's spawn a simple light bulb accessory as demonstration:

```python
#!/usr/bin/env python3

import os.path

from homekit import HomeKitServer
from homekit.model import Accessory, LightBulbService


if __name__ == '__main__':
    try:
        httpd = HomeKitServer(os.path.expanduser('~/.homekit/demoserver.json'))

        accessory = Accessory('Licht')
        lightService = LightBulbService()
        accessory.services.append(lightService)
        httpd.accessories.add_accessory(accessory)

        httpd.publish_device()
        print('published device and start serving')
        httpd.serve_forever()
    except KeyboardInterrupt:
        print('unpublish device')
        httpd.unpublish_device()
```

If everything went properly, you should be able to add this accessory to your home on your iOS device.

# HomeKit Controller

The following tools help to access HomeKit Accessories.

## discover.py

This tool will list all available HomeKit IP Accessories within the local network.

Usage:
```bash
python3 -m homekit.discover [-t ${TIMEOUT}]
```

The option `-t` specifies the timeout for the inquiry. This is optional and 10s are the default.

Output:
```
Name: smarthomebridge3._hap._tcp.local.
Url: http://192.168.178.21:51827
Configuration number (c#): 2
Feature Flags (ff): Paired (Flag: 0)
Device ID (id): 12:34:56:78:90:05
Model Name (md): Bridge
Protocol Version (pv): 1.0
State Number (s#): 1
Status Flags (sf): 0
Category Identifier (ci): Other (Id: 1)
```
Hints: 
 * Some devices like the Koogeek P1EU Plug need bluetooth to set up wireless before. Use your phone 
   or the proper app to perform this
 * paired devices should not show up

## identify.py

#TODO rework after identify.py was converted to controller

This tool will use the Identify Routine of a HomeKit IP Accessory.

Usage:

Identify an unpaired device by its id:
```bash
python3 -m homekit.identify -d ${DEVICEID} 
```

The option `-d` specifies the device id of the accessory to identify. Can be obtained via discovery.


Identify a paired device via controller file and alias:
```bash
python3 -m homekit.identify -f ${CONTROLLERFILE} -a ${ALIAS}
```

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.

Output:

Either a success message or an error message.

## pair.py

This tool will perform a pairing to a new accessory.

Usage:
```bash
python3 -m homekit.pair -d ${DEVICEID} -p ${SETUPCODE} -f ${PAIRINGDATAFILE} -a ${ALIAS} 
```

The option `-d` specifies the device id of the accessory to pair. Can be obtained via discovery.

The option `-p` specifies the HomeKit Setup Code. Can be obtained from the accessory.

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.


The file with the pairing data will be required to for any additional commands to the accessory.

## list_pairings.py

This tool will perform a query to list all pairings of an accessory.

Usage:
```bash
python3 -m homekit.list_pairings -f ${PAIRINGDATAFILE} -a ${ALIAS} 
```

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.

This will print information for each controller that is paired with the accessory:

```
Pairing Id: 3d65d692-90bb-41c2-9bd0-2cb7a3a5dd18
        Public Key: 0xed93c78f80e7bc8bce4fb548f1a6681284f952d37ffcb439d21f7a96c87defaf
        Permissions: 1 (admin user)
```

The information contains the pairing id, the public key of the device and permissions of the controller.

## unpair.py

This tool will remove a pairing from an accessory.

Usage:
```bash
python -m homekit.unpair -f ${PAIRINGDATAFILE} -a ${ALIAS}
```

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.

## get_accessories.py

This tool will read the accessory attribute database.

Usage:
```bash
python3 -m homekit.get_accessories -f ${PAIRINGDATAFILE} -a ${ALIAS} [-o {json,compact}]
```

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.

The option `-o` specifies the format of the output:
 * `json` displays the result as pretty printed JSON
 * `compact` reformats the output to get more on one screen

## get_characteristic.py
This tool will read values from one or more characteristics.

Usage:
```bash
python3 -m homekit.get_characteristic -f ${PAIRINGDATAFILE} -a ${ALIAS} -c ${Characteristics} [-m] [-p] [-t] [-e]
```

The option `-f` specifies the file that contains the pairing data.

The option `-a` specifies the alias for the device.

The option `-c` specifies the characteristics to read. The format is `<aid>.<cid>`. This 
option can be repeated to retrieve multiple characteristics with one call. 
 
The option `-m` specifies if the meta data should be read as well.

The option `-p` specifies if the permissions should be read as well.

The option `-t` specifies if the type information should be read as well.

The option `-e` specifies if the event data should be read as well.

## put_characteristic.py
This tool will write values to one characteristic.

Usage:
```bash
python3 -m homekit.put_characteristic -f ${PAIRINGDATAFILE} -c ${Characteristics} ${value}
```

The option `-f` specifies the file that contains the pairing data.

The option `-c` specifies the characteristics to change. The format is `<aid>.<cid> <value>`. This 
option can be repeated to change multiple characteristics with one call. 
 
For example, this command turns of a Koogeek P1EU Plug:
```
python3 -m homekit.put_characteristic -f koogeek.json -c 1.8 false
```

## get_events.py
This tool will register with an accessory and listen to the events send back from it.

Usage
```bash
python3 -m homekit.get_events -f ${PAIRINGDATAFILE} -c ${Characteristics} 
```

The option `-f` specifies the file that contains the pairing data.

The option `-c` specifies the characteristics to change. The format is `<aid>.<cid>`. This 
option can be repeated to listen to multiple characteristics with one call.

For example, you can listen to characteristics 1.8 (on characteristic), 1.22 (1 REALTIME_ENERGY) and 
1.23 (2 CURRENT_HOUR_DATA) of the Koogeek P1EU Plug with:
```bash
python3 -m homekit.get_events -f koogeek.json -c 1.8 -c 1.22 -c 1.23
```
This results in
```
event for 1.8: True
event for 1.22: 6.0
event for 1.23: 0.01666
event for 1.22: 17.0
event for 1.23: 0.06388
event for 1.23: 0.11111
event for 1.22: 18.0
event for 1.23: 0.16111
event for 1.8: False
```


# HomeKit Accessory

# Tests

The code was tested with the following devices by the author:
 * Koogeek P1EU Plug ([Vendor](https://www.koogeek.com/smart-home-2418/p-p1eu.html))

Users have tried (and succeeded, not checked by the author) to use the following devices:
 * Ikea TRÅDFRI ([Issue #13](https://github.com/jlusiardi/homekit_python/issues/13))
 * Philips Hue ([Issue #13](https://github.com/jlusiardi/homekit_python/issues/13))
 * Leviton DH6HD-1BZ ([Issue #16](https://github.com/jlusiardi/homekit_python/issues/16))
 * Lutron Caseta (Smart Bridge 2 / [Issue #17](https://github.com/jlusiardi/homekit_python/issues/17))
 * iHome iSP5 ([Issue #18](https://github.com/jlusiardi/homekit_python/issues/18))
