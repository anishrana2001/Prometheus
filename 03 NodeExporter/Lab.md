# NodeExporter

## We will going to install NodeExporter on workernode1 and master1 nodes.
## Node Exporter LAB : Note: NodeExporter service runs on 9100 port number.

### Create a user for node_exporter
```
useradd -M -r -s /bin/false node_exporter
```
### Direct link to download the NodeExporter.
```
https://prometheus.io/download/
```
### Download and install the packages. 
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```
### Check the file is correctly downloaded.
```
ls -ltr
```
### Untar the file.
```
tar xzf node_exporter-1.8.1.linux-amd64.tar.gz
```
### Go inside the directory.
```
cd node_exporter-1.8.1.linux-amd64/
```
### Copy the Node Exporter binary to the appropriate location and set ownership:
```
cp node_exporter /usr/local/bin/
```
### Assign the privileges to "node_exporter" user for node_exporter commands.
```
chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Create a systemd unit file for Node Exporter:

```
cat <<EOF>> /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
EOF
```

### As we have created a new service, thus, we have reload the dameon.
```
sudo systemctl daemon-reload
```
### Let's start the node_exporter service.
```
sudo systemctl start node_exporter
```
### It would be worth to enable the service, if system restart then our service will also be started.
```
sudo systemctl enable node_exporter
```
### Post checks are always a good practice. Hence, check the status of service.
```
systemctl status node_exporter
```
### If you observe that 9100 port is already in used then you may execute these 2 commands.
```
netstat -nlp | grep 9100
```
```
fuser -k 9100/tcp
```

### Now, you can run the below command to check if node_exporter is running as expected ?
```
/usr/local/bin/node_exporter
```

### Stop and disable the Firewall, if Firewall is raising some concern then you might need to stop the firewall. This is only for LAB. For production, you have to ask your Firewall team to open the desired ports.

```
systemctl stop firewalld
```
```
firewall-cmd --state
```
```
systemctl disable firewalld
```



### I am installing this "node_exporter" service on master1 node, thus, I can use localhost instead of my VM IP "192.168.1.31".
### Before adding the information on Prometheus server, let's check the target on Prometheus GUI. It should be only one.
```
http://192.168.1.31:9090/targets
```

### Configure Prometheus to Scrape Metrics. Locate the scrape_configs section and add a new entry under that section.
```
vi /etc/prometheus/prometheus.yml
-----------------

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node-exporter"                 # Added
    static_configs:                           # Added
      - targets: ["192.168.1.31:9100"]        # Added
```

### Reload the Prometheus config:
```
sudo killall -HUP prometheus
```
```
http://192.168.1.31:9090/targets
```

### Stop and disable the Firewall:
```
systemctl stop firewalld
```
firewall-cmd --state
```
systemctl disable firewalld
```
```
nc -vz 192.168.1.7 9100
```

### Go to Prometheus, http://192.168.1.31:9090

```
up
```
```
node_cpu_seconds_total
```
```
node_cpu_seconds_total{mode="steal"}
```
### For memory 
```
node_memory_MemFree_bytes
```
### For CPU
```
node_cpu_seconds_total
```

###Login to Node_exporter node
```
wget wget   https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/Packages/stress-ng-0.17.08-2.el9.x86_64.rpm
```
```
yum localinstall stress-ng-0.17.08-2.el9.x86_64.rpm -y
```
```
stress -m 2 
```
### m = memory 
### 2 = 2 virtual hogs


### Every time series is uniquely identified by its metric name. Metrics name specific to an application, the prefix is usually the application name itself. 
