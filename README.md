[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](http://standardjs.com/)

# DemoKit

ARTIK Cloud and ARTIK Module Demo

## Table of contents
* [Features](#features)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installing](#installing)
  * [Patch](#patch)
* [Create your own Alexa Voice Service](#create-your-own-alexa-voice-service)
* [Create your own ARTIK Cloud application](#create-your-own-artik-cloud-application)
* [Link the Alexa, ARTIK Cloud and Devices to DemoKit](#link-the-alexa-artik-cloud-and-devices-to-demokit)
* [Using the DemoKit](#using-the-demokit)
  * [Scenario mode](#scenario-mode)
  * [Custom mode (controlled by ARTIK Cloud Rule)](#custom-mode-controlled-by-artik-cloud-rule)
* [Open-source libraries/project used](#open-source-librariesproject-used)
* [License and Copyright](#license-and-copyright)

## Features

![diagram](/doc/diagram.png?raw=true)

### Master board
- Web Server (node.js)
  - DemoKit [setup page](https://192.168.0.10:9745)
  - OAuth2 with ARTIK Cloud
  - OAuth2 with AVS(Alexa Voice Service)
- AVS Client (node.js, AVS API v1)
  - MIC Recording --> Send PCM to AVS --> Receive MP3 from AVS --> Play MP3
  - Support Text->PCM->AVS for testing (SVOX pico2wave)
- Connect with ARTIK LCD Display
- Connect with BT Speaker
  - Play Dingdong mp3
- Play RTSP (GStreamer: gst-launch)
  - Video source from Slave board ARTIK Camera
  - Video source from Web cam
- Control Hue blub (node.js)
- Control Button & LED (node.js)
  - GPIO
- ARTIK Cloud proxy device
  - Demokit alexa-voice-service proxy
  - Demokit button proxy
  - Demokit mplayer proxy
  - Demokit rtsp proxy

### Slave board
- Connect with ARTIK Camera
- RTSP Server (GStreamer: test-launch)
- Control WeMo Switch (node.js, SSDP)
- Control Sliding door (node.js)
  - PWM
- Control Deadbolt
  - Power is controlled by WeMo
- Control Button & LED (node.js)
  - GPIO
- ARTIK Cloud proxy device
  - Demokit button proxy
  - Demokit sliding door proxy
  - Demokit front door proxy

## Getting Started

### Prerequisites

#### Hardware
- ARTIK 710 with LCD module
- ARTIK 710 with Camera module
- Belkin WeMo Insight Switch
- Hue Bridge + Hue Bulb
- ipTime A3004NS-d (Wireless router supporting Android phone usb tethering)
- Bluetooth Speaker
- Samsung Smartcam SNH-P6410BN

#### Software - Initial setup
- Download & Install OS image
  - Download from [ARTIK 710 Firmware Image A710-OS-2.0.0](https://developer.artik.io/documentation/downloads.html#firmware)
  - [Update guide](https://developer.artik.io/documentation/developer-guide/update-image/updating-artik-image.html)
- Install dependency packages to each ARTIK 710 boards
```sh
dnf update
dnf install git alsa-lib-devel npm gstreamer1-rtsp-server avahi-devel avahi-compat-libdns_sd-devel
```

- Install RTSP server program
```sh
dnf install autoconf automake m4 libtool gtk-doc glib2-devel gstreamer1-devel gstreamer1-plugins-base-devel
git clone https://github.com/GStreamer/gst-rtsp-server.git
cd gst-rtsp-server
git checkout 1.8
./autogen.sh
./configure --enable-examples
make
cd examples
libtool --mode=install install test-launch /usr/local/bin/
```

- Install TTS program (SVOX pico2wave)
```sh
[ubuntu-host]# apt-get source libttspico-utils
[ubuntu-host]# scp -r svox-1.0+git20130326/pico {your-710-board(MASTER)}
```
```sh
chmod +x autogen.sh
dnf install autoconf automake libtool popt-devel
./autogen.sh
./configure
make
make install
```
- Download dingdong MP3 file
  1. Open http://www.freesfx.co.uk/download/?type=mp3&id=16513
  1. Download MP3 file
  1. Copy to data/ding_dong_bell_door.mp3

#### Environment configuration

##### 710 board(master) IP settings

* Static MAC address setup (u-boot) Serial console
```sh
...
Hit any key to stop autoboot:  3   <-- Enter key 
artik710# run factory_load
artik710# factory_info list
artik710# factory_info write ethaddr <your mac address>  e.g.) 00:11:58:23:12:54
artik710# run factory_save
artik710# reset
```

* Static IP setup: 192.168.0.10 (OAuth2 callback url)
  1. Open the ipTime admin webpage (http://192.168.0.1)
  1. Setup static IP address using 710 board MAC address

##### BT pairing with 710 board(master)

* Setup from console or webpage (http://192.168.0.10:9745/bt)
```sh
bluetoothctl
[bluetooth]# scan on
[NEW] Device XX:XX:XX:XX:XX:XX Logitech X100
[bluetooth]# pair XX:XX:XX:XX:XX:XX
[bluetooth]# connect XX:XX:XX:XX:XX:XX
[bluetooth]# exit

pactl list cards
pactl set-card-profile 1 a2dp_sink
```

##### ipTime setup

1. Open the ipTime admin webpage (http://192.168.0.1)
1. Login (default id/password = 'admin' / 'admin')
1. Enable the USB tethering option

##### WeMo switch setup

1. Install WeMo app on your smartphone
1. Wi-Fi connect to WeMo (WeMo AP)
1. Run WeMo application
1. Setup WeMo to use ipTime AP

##### Hue bridge setup

1. Install Hue app on your smartphone
1. Add Hue Bridge on Hue app
1. Add Hue Bulb to Hue Bridge

##### Samsung Smartcam setup

1. https://www.samsungsmartcam.com/
1. Register your account
1. Register the SNH-P6410BN camera to your account
1. Setup camera id/password to 'admin' / '1234'

### Installing

* Master board (indoor simulation)
```sh
git clone https://github.com/SamsungARTIK/demokit
cd demokit
npm install

npm install -g pm2
pm2 start bin/www
pm2 startup
pm2 save

reboot
```
* Slave board (front door simulation)
```sh
git clone https://github.com/SamsungARTIK/demokit
cd demokit
npm install

npm install -g pm2
pm2 start slave.js
pm2 startup
pm2 save

reboot
```

* The **forever** and **forever-service** are no longer used due to bugs(does not kill child process).

<pre><code><del>npm install -g forever forever-service
forever-service install master --script bin/www
service master start
npm install -g forever forever-service
forever-service install slave --script slave.js
service slave start</del></code></pre>

### Patch

demokit/node_modules/mdns/lib/resolver_sequence_tasks.js
```javascript
        try {
-          //var error = dns_sd.buildException(errorCode);
+          var error = null;
          if ( ! error && service.interfaceIndex === iface) {
```

## Create your own Alexa Voice Service
1. https://developer.amazon.com/edw/home.html#/avs/list
1. Register a Product Type --> Device
1. Fill the form
   - Device Type Info
     - Company Name: ...
     - Device Type ID: demokit
     - Display Name: ...
   - Security profile
     - General
       - Client ID & Client Secret --> Copy to clipboard
     - Web settings
       - Allowed Return URLs: https://192.168.0.10:9745/avs/callback

## Create your own ARTIK Cloud application
1. https://developer.artik.cloud/dashboard/applications
1. Click NEW APPLICATION
1. Fill the form
   - APP NAME: DemoKit
   - DESCRIPTION: ...
   - AUTHORIZATION METHODS: Client credentials, auth code
   - AUTH REDIRECT URL: https://192.168.0.10:9745/akc/callback
1. Save
1. Permissions
   - PERMISSIONS
     - Profile: Check to Read
   - DEVICE PERMISSIONS
     - \+ SET PERMISSIONS FOR A SPECIFIC DEVICE
     - Belkin WeMo Insight Switch Proxy: Read + Write
     - Shell proxy: Read + Write
     - Demokit front door proxy: Read + Write
     - Demokit mplayer proxy: Read + Write
     - Demokit button proxy: Read + Write
     - Demokit alexa-voice-service proxy: Read + Write
     - Demokit rtsp proxy: Read + Write
1. Click 'DemoKit' from left panel
1. Click 'SHOW CLIENT ID & SECRET' from right side
   - Client ID & Secret --> Copy to clipboard

## Link the Alexa, ARTIK Cloud and Devices to DemoKit

### Setup Alexa Voice Service & ARTIK Cloud device token
- Open https://192.168.0.10:9745/setup
- Fill your AVS information to the client-id, client-secret field and 'Save'
- Fill your ARTIK Cloud application information to the client-client-secret field and 'Save'
- Click 'Get user tokens'
- Click 'Get device tokens' and 'Save'

### Register DemoKit to Hue bridge
- Press 'link button' on the Hue Bridge device
- Open https://192.168.0.10:9745/setup
- Click 'Get registered username' and 'Save'

## Using the DemoKit

### Scenario mode

- Change Demokit mode to scenario mode (https://192.168.0.10:9745/setup)
- Dingdong --> Press MIC btn --> say 'ask artik to unlock the front door' --> Release MIC btn
- Press MIC btn --> say 'ask artik to lock the front door' --> Release MIC btn

### Custom mode (controlled by ARTIK Cloud Rule)

- Change Demokit mode to custom mode (https://192.168.0.10:9745/setup)
- You can controll all modules(Dingdong, All buttons, Sliding door,...) from ARTIK Cloud Rules

## Open-source libraries/project used

### Web
- [Bootstrap](http://getbootstrap.com/) - Bootstrap is the most popular HTML, CSS, and JS framework for developing responsive, mobile first projects on the web.
- [Bootstrap Switch](http://www.bootstrap-switch.org/)
- [jQuery](https://jquery.com/) - jQuery is a fast, small, and feature-rich JavaScript library.
- [Notify.js](https://notifyjs.com/) - Notify.js is a jQuery plugin to provide simple yet fully customisable notifications.
- [socket.io](http://socket.io/) - Socket.IO enables real-time bidirectional event-based communication.
- [Font Awesome](http://fontawesome.io/) - The iconic font and CSS toolkit.

### Node.js
- [Express](http://expressjs.com/): [body-parser](https://www.npmjs.com/package/body-parser), [cookie-parser](https://www.npmjs.com/package/cookie-parser), [debug](https://www.npmjs.com/package/debug), [ejs](https://www.npmjs.com/package/ejs), [express](https://www.npmjs.com/package/express), [morgan](https://www.npmjs.com/package/morgan), [serve-favicon](https://www.npmjs.com/package/serve-favicon)
- [bl](https://www.npmjs.com/package/bl) - Buffer List: collect buffers and access with a standard readable Buffer interface, streamable too!
- [child-process-promise](https://www.npmjs.com/package/child-process-promise) - Simple wrapper around the "child_process" module that makes use of promises
- [dbus-native](https://www.npmjs.com/package/dbus-native) - D-bus protocol implementation in native javascript
- [http-message-parser](https://www.npmjs.com/package/http-message-parser) - HTTP message parser in JavaScript.
- [mdns](https://www.npmjs.com/package/mdns) - multicast DNS service discovery
- [mplayer](https://www.npmjs.com/package/mplayer) - Node.js wrapper for mplayer
- [mqtt](https://www.npmjs.com/package/mqtt) - MQTT.js is a client library for the MQTT protocol, written in JavaScript for node.js and the browser.
- [node-hue-api](https://www.npmjs.com/package/node-hue-api) - Phillips Hue API Library for Node.js
- [node-microphone](https://www.npmjs.com/package/node-microphone) - Allows Microphone access in node with arecord (Linux) and sox (Windows/OSX).
- [onoff](https://www.npmjs.com/package/onoff) - GPIO access and interrupt detection with JavaScript
- [passport](https://www.npmjs.com/package/passport) - Simple, unobtrusive authentication for Node.js.
- [passport-artikcloud](https://www.npmjs.com/package/passport-artikcloud) - ARTIK Cloud authentication strategy for Passport.
- [passport-oauth2](https://www.npmjs.com/package/passport-oauth2) - OAuth 2.0 authentication strategy for Passport.
- [passport-oauth2-refresh](https://www.npmjs.com/package/passport-oauth2-refresh) - A passport.js add-on to provide automatic OAuth 2.0 token refreshing.
- [pwm](https://www.npmjs.com/package/pwm) - Write to your PWM outputs
- [request](https://www.npmjs.com/package/request) - Simplified HTTP request client.
- [socket.io](https://www.npmjs.com/package/socket.io) - node.js realtime framework server
- [socket.io-client](https://www.npmjs.com/package/socket.io-client) - Realtime application framework (client)
- [websocket](https://www.npmjs.com/package/websocket) - Websocket Client & Server Library implementing the WebSocket protocol as specified in RFC 6455.
- [wemo](https://www.npmjs.com/package/wemo) - WeMo client

### Utils
- [RTSP server based on GStreamer](https://github.com/GStreamer/gst-rtsp-server)
- [SVOX TTS](http://packages.ubuntu.com/xenial/libttspico-utils)

### Etc
- [freeSFX](http://www.freesfx.co.uk/) - [Ding Dong Bell Door sound](http://www.freesfx.co.uk/download/?type=mp3&id=16513)

## License and Copyright

This project is licensed under the Apache-2.0 License - see the [LICENSE](LICENSE) file for details

Copyright (c) 2016 Samsung Electronics Co., Ltd.
