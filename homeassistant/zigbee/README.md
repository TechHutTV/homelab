## Navigation
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
  - [Dashboard](https://github.com/TechHutTV/homelab/tree/main/homeassistant/dashboard)
  - [LocalTuya](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya)
  - [__Zigbee__](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee)
    - [Seperate ZigbeeMQTT from Home Assistant in Proxmox](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee#seperate-zigbeemqtt-from-home-assistant-in-proxmox)
      - [Sonoff Zigbee 3.0 USB Dongle Plus V2 model "ZBDongle-E version"](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee#sonoff-zigbee-30-usb-dongle-plus-v2-model-zbdongle-e-version)
      - [ZigbeeMQTT and Mosquitto](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee#zigbeemqtt-and-mosquitto)
        - [Mosquitto MQTT Broker](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee#mosquitto-mqtt-broker)
        - [ZigbeeMQTT Configuration](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee#zigbeemqtt-configuration)

# Seperate ZigbeeMQTT from Home Assistant in Proxmox
Within Proxmox VE I have my ZigbeeMQTT and MQTT broker servers seperated from my Home Assistant virtual machine. I found I get slightly better preformance out of my Zigbee devices doing it this way, plus it's easier to backup and feed all my Zigbee devices to a new Home Assistant server if needed.

## Sonoff Zigbee 3.0 USB Dongle Plus V2 model "ZBDongle-E version"

This setup is centered around the Zigbee dongle that I purchased. I bought it due to it having decent reviews and it was on sale at the time of purchase. I have not tried anything else so I can't say if anything is better. __Due note__, it comes with outdated firmware that is not supported with current version of ZigbeeMQTT so you will need to update. Luckly, it's a fairly easy process.

* [YouTube video](https://www.youtube.com/watch?v=r0ihC8Dx3NM&) on how to update the firmware.
* Firmware: [https://github.com/darkxst/silabs-firmware-builder/tree/main/firmware_builds/zbdonglee](https://github.com/darkxst/silabs-firmware-builder/tree/main/firmware_builds/zbdonglee)
* Flashing Website: [https://darkxst.github.io/silabs-firmware-builder/](https://darkxst.github.io/silabs-firmware-builder/)
* Firmware I'm using as of writing this: [ncp-uart-hw-v7.4.5.0-zbdonglee-115200.gbl](https://github.com/darkxst/silabs-firmware-builder/blob/main/firmware_builds/zbdonglee/ncp-uart-hw-v7.4.5.0-zbdonglee-115200.gbl)

Once the firmware is updated you can plug it into your machine and find the ID and port using this command.
```bash
ls -l /dev/serial/by-id
```
This information will be used later in the ZigbeeMQTT configuration. You will get an output like this.
```bash
lrwxrwxrwx 1 root root 13 Dec 13 20:55 usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_20231007143933-if00 -> ../../ttyACM0
```
From there we can pull the following adapter locations
* Device ID: ```/dev/serial/by-id/usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_20231007143933-if00```
* USB Port: ```/dev/ttyACM0```

## ZigbeeMQTT and Mosquitto

For the inital set up I used a [Proxmox Community Helper Script](https://community-scripts.github.io/ProxmoxVE/scripts?id=zigbee2mqtt) to get this going. 
```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/zigbee2mqtt.sh)"
```
After running this command I chose the advanced setup to ensure the container is privilaged. I also set the IP address as static, but that's optional depending on your network setup. This helper script sets up a systemctl background service to automatically start ZigbeeMQTT when the system starts up. We are going to make some changes so let's ensure the serivce isn't running. Login to the console for our new LXC and run the command to stop the service.
```bash
sudo systemctl stop zigbee2mqtt
```
### Mosquitto MQTT Broker

For ZigbeeMQTT to be able to communicate with Home Assistant we will set up Mosquitto, a MQTT broker. You can set this up as a [Home Assistant addon](https://github.com/home-assistant/addons/blob/master/mosquitto/DOCS.md), it's [own seperate LXC](https://community-scripts.github.io/ProxmoxVE/scripts?id=mqtt), or many other deploylemt methods. I chose to install this on the same LXC that ZigbeeMQTT is running on. Lets install it and make sure it's running.

```bash
sudo apt install -y mosquitto
systemctl status mosquitto
```
Next we can use the built in passwork creation tool to set our password and encrypt it. Use the command below replacing _brandon_ with your username of choice.

```bash
mosquitto_passwd -c /etc/mosquitto/passwd brandon
```

You can run ```cat /etc/mosquitto/passwd``` to view the password encryption. Now lets ensure our configure is setup use nano to create or edit the file and copy the configuration below.

```bash
nano /etc/mosquitto/conf.d/default.conf
```
```yaml
allow_anonymous false
persistence true
password_file /etc/mosquitto/passwd
listener 1883
```
Now lets reload mosquitto and it will be ready to use in ZigbeeMQTT and Home Assistant.

systemctl restart mosquitto

### ZigbeeMQTT Configuration

Copied from [the official pro-install guide](https://github.com/community-scripts/ProxmoxVE/discussions/410), originally provided by the legend [tteck](https://github.com/tteck/Proxmox/discussions/2917). If you're using a different adapter, check out the official guide.

Edit the configuration here
```bash
nano /opt/zigbee2mqtt/data/configuration.yaml
```
Use this configuration if following my steps.
```yaml
frontend:
  port: 9442
homeassistant: true
permit_join: false
mqtt:
  base_topic: zigbee2mqtt
  server: 'mqtt://localhost:1883'
  user: brandon
  password: password
  keepalive: 60
  reject_unauthorized: true
  version: 4
serial:
  port: /dev/serial/by-id/usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_20231007143933-if00
  adapter: ember # for ITead Sonoff Zigbee 3.0 USB Dongle Plus V2 model "ZBDongle-E version"
advanced:
  pan_id: GENERATE
  network_key: GENERATE
  channel: 20
```
Be sure to change you username and password or mqtt and the device id under port. Once the configuration is saved run the service using npm so we can see if there are any errors.
```bash
cd /opt/zigbee2mqtt && npm start
```
If there are no errors you can exit the service with ctrl+c and restart the the service in the background using systemctl.

```bash
sudo systemctl restart zigbee2mqtt
```
Once everything is complete you can navigate to the frontend for ZigbeeMQTT and begin adding your devices! For my installation I navigate to ```http://10.0.0.106:9442```. Change your the IP to match yours. For my set up I added this to NGINX Proxy Manager so I just nagivate to ```https://zigbee.hopki.net/```
