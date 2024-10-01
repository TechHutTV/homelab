# Server Monitoring Stack
Docker compose featuring Grafana, Prometheus, Influxdb2, and Telegraf. Tested and working with the lastest version of Docker compose. This repository is a companion to the offical guide on our website. That is currently a work in progress. This is designed to be envolving repo as I add services and figure out better way to do this. 

**Note:** This is not a secure way to do this. ONLY use this in closed off, private testing enviorments. Eventually, we will add steps for SSL and other security features.

**System Specific Guides and Files:** [Unraid](https://github.com/TechHutTV/server-monitoring/tree/main/unraid) | Proxmox (wip) | Synology (wip)

## Installation
This assumes you have the last version of Docker. For my installation I used the [convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script) on Ubuntu 22.04.

When you clone the repository make sure it’s wherever you want some configuration files and scripts for services to be stored. Generally, I just do this in my user home directory.

```git clone https://github.com/TechHutTV/server-monitoring.git```

Now we can see how the included files are laid out:

```
tree server-monitoring/

server-monitoring/
├── compose.yaml
├── grafana
│   └── datasource.yml
├── prometheus
│   └── prometheus.yml
└── README.md

2 directories, 4 files
```
### compose.yaml
One thing we added is an extra_hosts section under a few of the services. This will enable they have access data exporters running on the host system outside of Docker by adding an entry to the container’s ```/etc/hosts``` file. If you have DNS set up for your LAN, then this part is not necessary. You can just substitute the DNS name as needed.

In the compose.yaml the I use the hostname “dockerlab” but replace this with yours throughout the guide. The important thing to note is that the data will be tagged with this name inside Prometheus as the machine name, and if you change it later, you’ll end up with multiple entries for the same machine. To check your hostname just run ```hostname``` in your terminal.
```
    extra_hosts:
      - "dockerlab:host-gateway"
```
The configuration files are within the repository we pull. You can just launch the compose file from there or move things around to meet your preferences.

### Configuration
Other than the docker compose file prometheus is the only one that needs to be configured before launching. 

```nano prometheus/prometheus.yml```

Adding a job for the Node Exporter or multiple node exporters can be done here. Use either the IP of the machine or a hostname if setup on your network or within your systems hosts file.

```
- job_name: node_exporter
  static_configs:
    - targets:
      - dockerlab:9100
      - 10.0.0.216:9100
      - unRAID:9100
```

Telegraf cant be configured until after we launch the stack for the first time and setup Influxdb2 as we will need our organization, username, token, and bucket. My default configuration is setup only to monitor docker with some system monitoring [plugins](https://docs.influxdata.com/telegraf/v1/plugins/) commented out. I recommened you read up on their [documentation](https://docs.influxdata.com/telegraf/v1/get-started/) for plugin specific configuration. To monitor docker with the included configuration you'll only need to change the information under the outputs plugin to match your setup
```
nano telegraf/telegraf.conf
```

```
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "yourtoken"
  organization = "yourorg"
  bucket = "localbucket"
```
### Launch the Stack

In our project director we can launch the stack using the compose up command.
```
docker compose up -d
```
If you run into any issues the logs are you best friend. Heck, check em anyway.
```
docker compose logs
```
To make configuration changes, simple save the edits you make and restart the stack.
```
docker compose restart
```
## Installing and Starting Node Exporter

The Prometheus node exporter is widely available as a package in the official repos of various Linux distros. To start we will be installing this Exporter on the same machine that we plan on running with Prometheus and Grafana. For Ubuntu and other Debian based distros, you can install it like this:

```sudo apt install prometheus-node-exporter```

While apt should start and enable the systemd service for the node exporter automatically, you can double check that like this:

```sudo systemctl status prometheus-node-exporter```

## Recommended Dashboards

The entire point of this is to have some pretty dashboards. These are the ones I found to work best with this setup. If needed I added the preferred language of the dashboards. as InfluxBD has multiple options when adding a new data connection.

[Docker Monitoring](https://grafana.com/grafana/dashboards/18389-influxdb-2-x-telegraf-docker-dashboard/) (Flux) - 18389

[Proxmox Monitoring](https://grafana.com/grafana/dashboards/10048-proxmox/) (InfluxQL) - 10048

[Unraid System Dashboard V2](https://grafana.com/grafana/dashboards/7233-unraid-system-dashboard-v2/) (InfluxQL) - 7233

[Node Exporter Full](https://grafana.com/grafana/dashboards/1860-node-exporter-full/) - 1860

#### Work in Progress
Feel free to send any feedback or rise any issues. This is a new repo and I'm organizing everything and trying to get it to be as good as I can.
