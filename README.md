# rpi-nightscout
CGM Remote Monitor aka [Nightscout](http://www.nightscout.info) for the Raspberry Pi 2/3/4.
This images offers a node.js webserver containing the Nightscout Application Version 14.2.5.

This image needs arm64 and therefore e.g. the ubuntu server os. This is because mongo 4.x is only
available for arm64 - not armv7(32bit) anymore.

# Docker Images
Docker images available here:
https://hub.docker.com/r/eyeoh/rpi-nightscout

# Provenance
This is a fork of:
https://github.com/dhermanns/rpi-nightscout

# Usage
This image requires ARM64-based hardware (not tested on Apple Silicon). 

Please follow the instructions available on the Nightscout site.

## Modify your Nightscout Settings
To modify your nightscout configuration, you can simply modify the environment settings in your docker-compose.yml file.
E.g. to modify the default alarm ranges change the values of BG_HIGH, BG_LOW, etc.:
```
nightscout:
  image: eyeoh/rpi-nightscout:14.2.5
  environment:
    TZ: Europe/Berlin
    MONGO_CONNECTION: mongodb://mongo:27017/nightscout
    API_SECRET: nightscout2000
    BG_HIGH: 220
    BG_LOW: 60
    BG_TARGET_TOP: 180
    BG_TARGET_BOTTOM: 80
```

## Fire-up the Nightscout Application
After cloning you are ready to start the Nightscout Application using docker compose
```
docker-compose up -d
```

## Access your Nightscout Application
Now you should be able to connect to your Nightscout Application by entering
```
http://<name-of-my-raspberry>:1337
```
in your Webbrowser.

## Optional: Direct access to the mongo database
If you would like direct access to the mongo database, you will have to open up the mongo db port to be accessible from outside the docker container. You can easily do this by modifying the `docker-compose.yml`:
```
mongo:
  image: mongo:4.4.9
  ports:
    - "27017:27017"
    - "27018:27018"
    - "27019:27019"
    - "28017:28017"
  restart: always
```

This way you should be able to use other tools that would like direct access.

## Activate HTTPS
If you would like to use e.g. the MyBG Apple Watch App, you will have to activate HTTPS for the nightscout cgm remote monitor.
To do this, first create a private key and a certification request for your node-server:
```
openssl req -nodes -newkey rsa:2048 -keyout server.key -out server.csr
```
Afterwards grab a free SSL-Certificate e.g. from this Certification Agency:
```
https://buy.wosign.com/free/
```
Download your server certificate and use the certificates contained in "for Other Server.zip".
Create a certification chain in one file:
```
cat 3_user_my-server-domain.de.crt 2_issuer_Intermediate.crt 1_cross_Intermediate.crt > serverchain.crt
```
If you don't create the chain of trust in this way, your browser will complain about your certificate and the iOS and Apple Watch Apps won't work!

Then configure your docker-compose.yml and activate node to use your newly created certificates:
```
nightscout:
  image: dhermanns/rpi-nightscout:0.9.2
  environment:
    SSL_KEY: /var/opt/ssl/server.key
    SSL_CERT: /var/opt/ssl/serverchain.crt
```
I don't needed to add anything to the SSL_CA environment value. Take care that you change the URL in the uploader to HTTPS. Change the URL in all other devices e.g. Watches or Smartphones.

License
---------------

[agpl-3]: http://www.gnu.org/licenses/agpl-3.0.txt

    cgm-remote-monitor - web app to broadcast cgm readings
    Copyright (C) 2015 The Nightscout Foundation, http://www.nightscoutfoundation.org.

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published
    by the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
