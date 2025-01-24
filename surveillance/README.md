# Frigate NVR in Proxmox with Coral TPU

This is something I am still trying to learn on figure out. My configuration is far from perfect as many of the features I want are not working or set up yet.

## To-Do
* Stream audio from cameras in live view
* Enable two-way audio for supported cameras
* Configure zones for event capture

## LXC Setup
Install docker as priv [https://community-scripts.github.io/ProxmoxVE/scripts?id=docker](https://community-scripts.github.io/ProxmoxVE/scripts?id=docker)

```
# USB passthrough
lxc.cgroup2.devices.allow: a
lxc.cap.drop:
lxc.cgroup2.devices.allow: c 188:* rwm
lxc.cgroup2.devices.allow: c 189:* rwm
lxc.mount.entry: /dev/serial/by-id  dev/serial/by-id  none bind,optional,create=dir
lxc.mount.entry: /dev/ttyUSB0       dev/ttyUSB0       none bind,optional,create=file
lxc.mount.entry: /dev/ttyUSB1       dev/ttyUSB1       none bind,optional,create=file
lxc.mount.entry: /dev/ttyACM0       dev/ttyACM0       none bind,optional,create=file
lxc.mount.entry: /dev/ttyACM1       dev/ttyACM1       none bind,optional,create=file
# Coral M.2 TPU
features: nesting=1
lxc.mount.entry: /dev/apex_0 dev/apex_0 none bind,optional,create=file
# VAAPI hardware transcoding
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
```

## Install Coral AI PCIe Accelerator Drivers
https://coral.ai/docs/m2/get-started/#2a-on-linux
reboot

Check for devices

```
lspci -nn | grep 089a
```

Two Coral TPU’s Found

Check for Apex devices

```
ls /dev/apex_*
```

---

## Install MQTT (Mosquitto)
I don't currently run my MQTT server on the same LXC as Frigate, but you can install it on the same machine with no issues. 

```
sudo apt install mosquitto mosquitto-clients

sudo systemctl is-enabled mosquitto

sudo systemctl status mosquitto
```


Edit the MQTT config file.

nano /etc/mosquitto/mosquitto.conf

Add these two lines

```
allow_anonymous true
listener 1883
```
https://dgpu-docs.intel.com/driver/installation.html
systemctl restart mosquitto

Needs updated: [https://docs.vultr.com/install-mosquitto-mqtt-broker-on-ubuntu-20-04-server](https://docs.vultr.com/install-mosquitto-mqtt-broker-on-ubuntu-20-04-server)

## Stream audio from cameras in live view
Add ffmpeg codec under the go2rtc entry for the camera.
```
go2rtc:
  streams:
    kitchen:
      - rtsp://admin:password@10.0.0.103/Preview_01_main
      - "ffmpeg:living_room#audio=opus"
``` 
```
  Kitchen:
    ffmpeg:
      inputs:
        - path: rtsp://localhost:8554/kitchen
          input_args: preset-rtsp-restream
          roles:
            - record
        - path: rtsp://localhost:8554/kitchen_sub
          input_args: preset-rtsp-restream
          roles:
            - detect
    ...
    live:
      stream_name: kitchen
```
