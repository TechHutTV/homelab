# Monitoring Unraid with Telegraf

## Add your configuration
Before we add our Telegraf container, we need to add our configuration. In this example, we are placing it in the default directory for Docker configurations on Unraid. Change anything here to better fit your setup. When working with my appdata directory in Unraid, I generally use the [Dynamix File Manager](https://forums.unraid.net/topic/120982-dynamix-file-manager/), It's an awesome tool that makes navigating shares in the Unraid UI easy.

First, download the [telegraf.conf](https://github.com/TechHutTV/server-monitoring/blob/main/unraid/telegraf.conf) file located in this repository and make any changes you'd like. Most everything in my configuration should work as-is with Unraid, but you will need to edit the output plugin information to properly send data to your InfluxDB 2 bucket. _Note: Use a new bucket specifically for this Unraid instance._

```
[[outputs.influxdb_v2]]
  urls = ["http://ip:8086"]
  token = "yourtoken"
  organization = "yourorg"
  bucket = "unraidbucket"
```

In your appdata directory on Unraid make a new folder called telegraf. Within your telegraf folder, upload the edited telegraf.conf file. The file path will look like this if you are following the default Unriad setup. ```/mnt/user/appdata/telegraf/telegraf.conf ```


## Setting up the conatiner

Use the official Telegraf container from the Applications page. We will be using the [golift/telegraf](https://hub.docker.com/golift/telegraf) in place of the office image as it includes some packages needed for full monitoring of the hardware.
### Change the following:

Repository: ```golift/telegraf```

Registry URL: ```https://hub.docker.com/golift/telegraf```

This container provides a telegraf docker image with added tools for monitoring disks, sensors, and IPMI. This exists because the base Telegraf Docker image makes it *very* difficult to monitor some system metrics. Applications added: smartctl (smartmontools), ipmitool, nvme-cli, sensors (lm-sensors), mtr (mtr-tiny), sudo. Sudoers entries are added for smartctl, ipmitool, and nvme.

### Add the following:

Extra Parameters: ```/bin/bash -c "/entrypoint.sh telegraf"```

Extra Argument: ```--user telegraf:$(stat -c '%g' /var/run/docker.sock)```

### Start and verify

Other than these adjustments, everything can be left as is included in the template. Check over everything and click Apply. Go to your Docker page in Unraid and check to see if the container is running. Check the log files to ensure there are no issues and that all the plugins are properly active.

If there are no issues, we check to see if the data is being properly exported to InfluxDB 2. Head over to your dashboard and open the bucket you created for Unraid. It should look like the picture below. Go through the data and make sure nothing is missing. In my installation, I have 7 tags for Docker data under the _messurements ID and another tag for every plugin I have enabled.

![Unraid data preview in InfluxDB 2](https://github.com/TechHutTV/server-monitoring/blob/main/unraid/unraid-data-preview.png?raw=true)

## Nivida Support

I have not been able to test this, but feel free to. The plugin for this is commented out in telegraf.conf. If you want to use a NVidia card, you must first [install the drivers](https://forums.unraid.net/topic/98978-plugin-nvidia-driver/) thanks to [@ich777](https://forums.unraid.net/profile/72388-ich777/).

```
[[inputs.nvidia_smi]]
  bin_path = "/usr/bin/nvidia-smi"
  timeout = "15s"
```
To get this working, an extra argument is needed in the Docker template for Telegraf on Unraid.

Extra Argument: ```--runtime=nvidia```

Create a custom user script that should be executed during startup with the following content:

```
#!/bin/bash
nvidia-persistenced
```

## To-Do
- [x] Create a working configuration.
- [ ] Create a custom Unraid template for golift/telegraf.
- [ ] Test inputs.apcupsd and add steps.
- [ ] Test and verify inputs.nvidia_smi steps.
- [ ] Add steps on enabling SSL for better security.
- [ ] Add steps for connecting this data to Grafana.
- [ ] Add tested and recommended Unraid dashboards.

I'm more than open to any suggestions and improvements!
