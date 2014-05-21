# PyDoorbell

## Connecting to your Raspberry Pi via serial

Connect serial cable to the outer layer of pins:

    - Black wire = 3rd pin in
    - White wire = 4th pin in
    - Green wire = 5th pin in

Insert USB.

Run:

    sudo screen /dev/ttyUSB0 115200

N.B. 115200 is the connection speed or [baud rate](https://en.wikipedia.org/wiki/Baud)

Log in to the Raspberry Pi:

user: `pi`
password: `raspberry`


## Connecting to your Raspberry Pi via WiFi

Use `sudo ifconfig -a` to find out `HWaddr`, e.g.
MAC adress:

Matt's Pi:  HWaddr 80:1f:02:9b:f5:5c

Edit `/etc/network/interfaces` to set up `wlan 0`

    auto wlan0
    allow-hotplug wlan0
    iface wlan0 inet dhcp
            wpa-ssid "<network SSID>"
            wpa-psk "<password>"

Restart network interface:

   sudo service networking stop
   sudo service networking start