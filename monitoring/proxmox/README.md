## Proxmox Community Helper Scripts

### Setup InfluxDB2

influxdb
```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/influxdb.sh)"
```
Make sure you select version 2 of InfluxDB when installing. You can add Telegraf if you want, but I don't find this nessesary as I will be using the Proxmox Metric server to send data on all the containers.

Now, navigate to http://IP:8086/ to begin the setup process. For the initial bucket I just named it default as I will make seperate buckets for various services. Grab your API token and save it somewhere safe.

From here click on advanced and we're going to create a new bucket for proxmox. I'm calling this bucket proxmox and keeping the data for 90 days.

### Metric Server on Proxmox

Getting this setup is easy, Head over the your instance of Proxmox VE and under Database scroll to the bottom until you reach Metric Server. Click on Add and then InfluxDB. From here input your server IP, switch it to HTTP or HTTPS if you setup a proxy, and all the appropreate information including the API token we saved earlier. 

### Visualize with Grafana 

grafana
```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/grafana.sh)"
```
Navigate to http://IP:3000/ and login with username admin and password admin. Then set your new password.

Within Grafana head to Connections> Add new connection > and add InfluxDB with your creditentials. Also, be sure to change the laungage from InfluxQL to Flux. When you click save and test you should get the message "datasource is working. 4 buckets found".

Dashbaord I'm using: 15356
