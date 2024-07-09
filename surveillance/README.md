future frigate setup

Fresh Ubuntu Install

Install Coral AI PCIe Accelerator Drivers
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

Install MQTT (Mosquitto)

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

