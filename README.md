# What is this?

This is a server (NodeJS) that connect to Risco Cloud Alarm Security System.
Integration works only when proper Ethernet module is added to your Risco Unit and you are able to arm & disarm your system via https://www.riscocloud.com/ELAS/WebUI.

The purpose of this service was to separate Risco API endpoints from Homebridge. During my tests Homebridge was quite unresponsive and prone to strange behaviours / delays. If you're already have a hardware where you run Homebridge, you can also install separate service for Risco Alarm, that handle all logic inside and expose only http endpoint where from Homebridge / any other platform you can read / arm / disarm your Risco. Polling is enabled by default.

## Available methods

- GET /alarm/check - get Risco Status (3 - disarmed, 1 - armed)
- GET /alarm/arm/:state - set :state to away to arm your Risco, set to any other to disarm.
- GET /alarm/bypass/:id/state - set sensor with provided id to be omitted during next arm cycle
- GET /alarm/bypass/:id - get bypass value for requested sensor

## RassberyPi how to start service automatically

For this we should create new service in /lib/systemd/system/
Instructions can be found here:
https://www.paulaikman.co.uk/nodejs-services-raspberrypi/

### Service control

sudo systemctl start|stop|restart risco.service
sudo systemctl restart risco.service

## Cofnig file

Before you run this service, make sure you update config.json file.

```json
  "RISCO_USER":   Riso User, usually your email address
  "RISCO_PASS":   Your Password to Risco system
  "RISCO_PIN":    Your PIN
  "RISCO_SITEID": Your Risco SideID
```

### How to get your riscoSiteId

To get your riscoSiteId, login to riscocloud via ChromeBrowser (first login screen), and before providing your PIN (second login page), display source of the page and find string: <div class="site-name" ... it will look like:

```html
<div class="site-name" id="site_12345_div"></div>
```

In that case "12345" is your siteId which should be placed in new config file.

## Use with Homebridge

I use this service with Http-SecuritySystem npm package dedicated for Homebridge
https://www.npmjs.com/package/homebridge-http-securitysystem

My config for this plugin:

```json
  "accessory": "Http-SecuritySystem",
  "name": "Home security",
  "debug": false,
  "username": "",
  "password": "",
  "immediately": false,
  "polling": true,
  "pollInterval": 10000,
  "http_method": "GET",
  "urls": {
      "stay": {
          "url": "http://localhost:8889/alarm/arm/stay",
          "body": "stay"
      },
      "away": {
          "url": "http://localhost:8889/alarm/arm/away",
          "body": "away"
      },
      "night": {
          "url": "http://localhost:8889/alarm/arm/night",
          "body": "night"
      },
      "disarm": {
          "url": "http://localhost:8889/alarm/arm/disarm",
          "body": ""
      },
      "readCurrentState": {
          "url": "http://localhost:8889/alarm/check",
          "body": ""
      },
      "readTargetState": {
          "url": "http://localhost:8889/alarm/check",
          "body": ""
      }
  }

```
