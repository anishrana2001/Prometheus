# Prometheus

### Create a user for prometheus
#### To create users without their home directories, the '-M' option is used.
#### To see all options just execute the command "useradd" on the terminal.
```
useradd -M -r -s /bin/false prometheus
```
### Create a new directories, one is for Prometheus, where we can store the prometheus configuration files and 2nd is for Prometheus storage.
```
mkdir /etc/prometheus /var/lib/prometheus
```
### After that, create a directory where we can download our Prometheus packages. So that it will be easy for us when we will going to un-install the Prometheus server.
```
mkdir /data/
```
 ### Let's go inside this directory.
 ```
cd /data/
```
### Download the latest packages.
```
https://prometheus.io/download/
```

### copy the link "prometheus-2.52.0.linux-amd64.tar.gz"
### Execute the command on shell.
```
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
```

file prometheus-2.52.0.linux-amd64.tar.gz

### Untar the file.
```
tar xvf prometheus-2.52.0.linux-amd64.tar.gz 
```
### Go inside the directory.
```
cd prometheus-2.52.0.linux-amd64/
```
```
ls -ltr
```

In this directory, you will observe 2 files, i.e. prometheus and promtool. Copy both files in the "/usr/local/bin/" directory. So that we can run these commands without specifying the absolute path.
```
cp prometheus promtool /usr/local/bin/
```
### Assign the privileges to prometheus user for both commands.
```
chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
```
### One can observe 2 directries, consoles,console_libraries. Copies these directories into our newly created directory "/etc/prometheus/".
```
cp -r {consoles,console_libraries}  /etc/prometheus/
```
### Now, it's time to copy the config file of prometheus into directory "/etc/prometheus/".
```
cp prometheus.yml /etc/prometheus/
```
### Modify the ownership of directory "/etc/prometheus/" to prometheus.
```
chown -R prometheus:prometheus /etc/prometheus
```
### Modify the ownership of storage directory "/var/lib/prometheus" of Prometheus to prometheus user.
```
chown prometheus:prometheus /var/lib/prometheus
```

### Now, if we execute the below command, you will observe that your prometheus application is running. 
```
prometheus --config.file=/etc/prometheus/prometheus.yml
```
### We can also create a Systemctl service and then enable this service, thus, in futher if our server OS restarted  then our service will be started by systectl service.
```
cat <<EOF>> /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Time Series Collection and Processing Server
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Start and enable the Prometheus service:
Make an HTTP request to Prometheus to verify it is able to respond:
You can also access Prometheus in a browser using the server's public IP address: http://192.168.1.31:9090.
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
 --config.file /etc/prometheus/prometheus.yml \
 --storage.tsdb.path /var/lib/prometheus/ \
 --web.console.templates=/etc/prometheus/consoles \
 --web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
EOF
```
### As we have created a new service, thus, we have reload the daemon.
```
sudo systemctl daemon-reload
```
### Let's start the prometheus service.
```
sudo systemctl start prometheus
```
### It would be worth to enable the service, if system restart then our service will also be started.
```
sudo systemctl enable prometheus
```
### Post checks are always a good practice. Hence, check the status of service.
```
systemctl status prometheus
```

### For more troubleshooting, you may execute below commands. Only to see more logs.
```
journalctl -eu prometheus
```

### Post checks.
```
curl localhost:9090
```

### Open the web portal "http://192.168.1.31:9090" Please bear in mind that Prometheus will 


### Log in to your Prometheus server. Edit the Prometheus configuration file "sudo vi /etc/prometheus/prometheus.yml"

```
---------------------------------
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']   # If you wish, you can change it to VM IP, for me its 192.168.1.31
 -----------------------------------
```

### There are three blocks of configuration in the example configuration file: global, rule_files, and scrape_configs.
### In global block, We have two options present. The first, scrape_interval, controls how often Prometheus will scrape targets. You can also override this for individual targets. In this case the global setting is to scrape every 15 seconds. The evaluation_interval option controls how often Prometheus will evaluate rules. Prometheus uses rules to create new time series and to generate alerts. In this case, it will check the new rules every 15 seconds.

### We can also check the syntax of prometheus.yaml file.
```
promtool check config /etc/prometheus/prometheus.yml
```
### Reload the Prometheus configuration: 

```
killall -HUP prometheus
```
### or 
```
systemctl restart prometheus
```
### My VM (master1.example.com) has an IP address 192.168.1.31 and this VM is running on my Laptop. So, I will use my VM IP. If you are using cloud VM then you can also use "localhost" instead of IP address.
```
http://192.168.1.31:9090/api/v1/status/config
```
```
http://192.168.1.31:9090/metrics
```
### one metric that Prometheus exports about itself is called "promhttp_metric_handler_requests_total" (the total number of /metrics requests the Prometheus server has served). 
### Go ahead and enter this into the expression console:

```
http://192.168.1.31:9090
```

```
promhttp_metric_handler_requests_total
```
