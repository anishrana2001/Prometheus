### Primary prometheus server
### allow root user to login from ssh connection

```
vi /etc/ssh/sshd_config
```
```
PermitRootLogin yes
```

```
service sshd restart
```

### Now, login into the Workernode1 server.
```
cat <<EOF>> /data/prometheus-HA.sh
prometheus_PIP=192.168.1.31
echo -e "\033[32m create a user 'prometheus'\033[m"
useradd -M -r -s /bin/false prometheus

echo -e "\033[32m Creating '/etc/prometheus' directory \033[m"
echo "mkdir /etc/prometheus /var/lib/prometheus"
mkdir /etc/prometheus /var/lib/prometheus

echo "chown prometheus:prometheus /var/lib/prometheus"
chown prometheus:prometheus /var/lib/prometheus

echo -e "\033[32m Creating '/etc/prometheus/rules' directory \033[m"
echo "mkdir -p /etc/prometheus/rules"
mkdir -p /etc/prometheus/rules


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
echo -e "\033[32m Now, you need to execute the below commands for copy the Primary server (Prometheus configuration and rules files) on this server. \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32mprometheus_PIP=192.168.1.31 \033[m"
echo -e "\033[32mscp root@$prometheus_PIP:/etc/prometheus/prometheus.yml /tmp \033[m"
echo -e "\033[32mmv /tmp/prometheus.yml /etc/prometheus/prometheus.yml  \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"

echo -e "\033[32mscp root@$prometheus_PIP:/etc/prometheus/rules/* /etc/prometheus/rules/  \033[m"
echo -e "\033[32mchown -R prometheus:prometheus /etc/prometheus \033[m"
echo -e "\033[32m --- \033[m"
echo -e "\033[32m --- \033[m"

EOF
```


### Execute the script.
```
chmod 755 /data/prometheus-HA.sh 
sh /data/prometheus-HA.sh 
```

### Check the secondary Prometheus configuration.
```
promtool check config /etc/prometheus/prometheus.yml
```

### Start the Prometheus service
```
systemctl start prometheus.service
```

### Check the status.

```
systemctl status prometheus.service
```

### Enable the Prometheus service

```
systemctl enable prometheus.service
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
http://192.168.1.32:9090
```
```
http://192.168.1.32:9090/rules
```
