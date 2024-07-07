# PushGateway

### How to install Pushgateway?

### Create a user and group for Pushgateway
```
sudo useradd -M -r -s /bin/false pushgateway
```

### Create a directory, where we will download our packages.
```
mkdir /data
```
```
cd /data
```
###  Open the Prometheus URL:  https://prometheus.io/download/
```
wget https://github.com/prometheus/pushgateway/releases/download/v1.9.0/pushgateway-1.9.0.linux-amd64.tar.gz
```
### Untar the file.
```
tar xvf pushgateway-1.9.0.linux-amd64.tar.gz
```
### Go inside this directory.
```
cd pushgateway-1.9.0.linux-amd64/
```
### Copy the binary file into the "/usr/local/bin/" directory.
```
cp pushgateway /usr/local/bin/
```
### Giviing the permission. 
```
chown pushgateway:pushgateway /usr/local/bin/pushgateway
```
### Create a systemd unit file for Pushgateway:

```
cat <<EOF>> /etc/systemd/system/pushgateway.service
[Unit]
Description=Prometheus Pushgateway
Wants=network-online.target
After=network-online.target
[Service]
User=pushgateway
Group=pushgateway
Type=simple
ExecStart=/usr/local/bin/pushgateway
    --web.listen-address=":9091" \
    --web.telemetry-path="/metrics" \
    --persistence.file="/tmp/metric.store" \
    --persistence.interval=1m \
    --log.level="info" \
    --log.format="logger:stdout?json=true"
[Install]
WantedBy=multi-user.target
EOF
```
###  Start and enable the pushgateway service:
```
sudo systemctl enable pushgateway
```
```
sudo systemctl start pushgateway
```



###  Verify the service is running and it is serving metrics:
```
systemctl status pushgateway
```
```
curl localhost:9091/metrics
```
### check the machine IP address
```
 ip a
```
### My VM IP 192.168.1.33


### Login into the Prometheus server and configure Prometheus to Scrape Metrics from Pushgateway
### Edit the Prometheus config:
```
vi /etc/prometheus/prometheus.yml

Under the scrape_configs section, add a scrape configuration for Pushgateway. Be sure to set honor_labels: true:

- job_name: 'Pushgateway'
 honor_labels: true
 static_configs:
 - targets: ['192.168.1.33:9091']
 ```
 

###  Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

###  Restart Prometheus to load the new configuration:
```
killall -HUP prometheus
```

###  Use the expression browser to verify you can see Pushgateway metrics in Prometheus.
```
http://192.168.1.31:9090/targets?search=
```
```
http://192.168.1.31:9090
```
```
up
pushgateway_build_info
```

###  How to push the data to pushgateway so that Prometheus will scrape the data.
###  With the help of Bash and Python script.

###  bash -- "key value", in metric "web_app2" we have defined the lables too. See the at the end.
```
echo "web_app1 0" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/web_app1/
echo "web_app2 10.03" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/web_app2/instance/192.168.1.50:9000/cpu/0
echo "web_app2 12.03" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/web_app2/instance/192.168.1.50:9000/cpu/1
```
```
cat << EOF | curl --data-binary @- http://192.168.1.33:9091/metrics/job/datacenter_delhi/instance/my_instance1
# TYPE temperature gauge
temperature{datacenter="delhi_state1"} 21
temperature{database="delhi_state2"} 20
# TYPE my_metric gauge
# HELP my_metric An example.
my_new_metric 3
EOF
```

### Open the Prometheus GUI 

```
http://192.168.1.31:9090
```

```
web_app1
```
```
web_app2
```
