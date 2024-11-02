# Frigate NVR

This is something I am still trying to learn on figure out. My configuration is far from perfect as many of the features I want are not working or set up yet.

## To-Do
* Stream audio from cameras in live view
* Enable two-way audio for supported cameras
* Configure zones for event capture

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
