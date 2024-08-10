

### Login into the Workernode2 server (192.168.1.33).
```
cat <<EOF>> prometheus-federation.sh
echo -e "\033[32m create a user 'prometheus'\033[m"
useradd -M -r -s /bin/false prometheus

echo -e "\033[32m Creating '/etc/prometheus' directory \033[m"
echo "mkdir /etc/prometheus /var/lib/prometheus"
mkdir /etc/prometheus /var/lib/prometheus

echo "chown prometheus:prometheus /var/lib/prometheus"
chown prometheus:prometheus /var/lib/prometheus

echo -e "\033[32m Download the file\033[m"
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"

echo -e "\033[32m Extract the .tar file.\033[m"
echo "tar xvf prometheus-2.52.0.linux-amd64.tar.gz"
tar xvf prometheus-2.52.0.linux-amd64.tar.gz 
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m Go inside the directory  \033[m"
echo "cd prometheus-2.52.0.linux-amd64/"
cd prometheus-2.52.0.linux-amd64/
echo -e "\033[32m Copy the binary files  \033[m"
cp prometheus promtool /usr/local/bin/
echo "cp prometheus promtool /usr/local/bin/"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m Changes the owernership of both files (prometheus,promtool) \033[m"
echo "chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}"
chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}

echo -e "\033[32mCopy the Prometheus configuration file on "/etc/prometheus/" \033[m"
echo "cp prometheus.yml /etc/prometheus/"
cp /root/prometheus-2.52.0.linux-amd64/prometheus.yml /etc/prometheus/

echo -e "\033[32mChange the ownership of Prometheus' configuration file  "/etc/prometheus/prometheus.yml" \033[m"
echo "chown -R prometheus:prometheus /etc/prometheus"
chown -R prometheus:prometheus /etc/prometheus


echo -e "\033[32m Copy the files consoles & console_libraries into the directory '/etc/prometheus/'  \033[m"
echo "cp -r {consoles,console_libraries}  /etc/prometheus/"
cp -r {consoles,console_libraries}  /etc/prometheus/

echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"

echo -e "\033[32m Creating the Prometheus service \033[m"
echo "[Unit]
Description=Prometheus Time_Series_Collection_&_Processing service
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
 --config.file /etc/prometheus/prometheus.yml \
 --storage.tsdb.path /var/lib/prometheus/ \
 --web.console.templates=/etc/prometheus/consoles \
 --web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/prometheus.service

echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"

echo -e "\033[32m Starting the Prometheus services \033[m"
echo "systemctl start prometheus.service"
systemctl start prometheus.service

echo -e "\033[32m Enabling the Prometheus service \033[m"
echo "systemctl enable prometheus.service"
systemctl enable prometheus.service
EOF
```


### Execute the script.
```
sh prometheus-federation.sh
```

### Check the Federate Prometheus configuration.
```
promtool check config /etc/prometheus/prometheus.yml
```

### Check the status.

```
systemctl status prometheus.service
```

### Let's check the Prometheus web portal
```
curl localhost:9090
```

### Check the Prometheus metrics CLI 
```
curl localhost:9090/metrics
```
### Let's open the web URL 

```
http://192.168.1.33:9090
```
```
http://192.168.1.33:9090/rules
```

### Our new Prometheus server is up and running and now, we can add the federation configuration

```
vi /etc/prometheus/prometheus.yml 
```

```
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true    # It preserve the original labels of the data that we are pulling.
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'


    static_configs:
      - targets:
        - '192.168.1.31:9090'
        - '192.168.1.32:9090'
```


### Check the configuration. 
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload the configuration.
```
killall -HUP prometheus
```
