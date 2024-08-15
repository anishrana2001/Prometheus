# How to uninstall the monitoring tools
## 01. Uninstall the Prometheus federation
## 02. Uninstall the Prometheus HA server from Workernode1.
## 03. Uninstall the Alertmanager HA server from Workernode2.
## 04. Uninstall the Alertmanager server from Workernode1.
## 05. Uninstall / remove HTTP Service Discover.
## 06. Uninstall the pushgateway from workernode2.
## 07. Uninstall the NodeExporter from nodes (master and workernode1)
## 08. Uninstall the Prometheus from master node.
# 
# 


## 01. Uninstall the Prometheus federation server from wokernode2 (192.168.1.33) , 
### 
### A simple script has been developed to uninstall this Prometheus federation server. In this script, 
#### It first stop the Prometheus service.
#### Disable the Prometheus service
#### Remove the servie, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### Finally remove the Prometheus user and federation script that we used to install Prometheus Federation. 
```
cat <<EOF>> /data/uninstall-prometheus-federation.sh
systemctl stop prometheus.service
systemctl disable prometheus.service
rm -rf /etc/systemd/system/prometheus.service
rm -rf /etc/prometheus/*
rmdir /etc/prometheus
rm -rf /var/lib/prometheus/*
rmdir /var/lib/prometheus
rm -rf /usr/local/bin/promtool/*
cd /data
rm -rf /data/prometheus-2.52.0.linux-amd64.tar.gz
rm -rf /data/prometheus-2.52.0.linux-amd64/*
rmdir /data/prometheus-2.52.0.linux-amd64/
userdel prometheus
rm -rf /data/prometheus-federation.sh
EOF
```
### Giving the right permission and then execute the script.
```
chmod 755 /data/uninstall-prometheus-federation.sh
sh /data/uninstall-prometheus-federation.sh
```
### Remove this script too.
```
rm -rf /data/uninstall-prometheus-federation.sh
```





## 02. Uninstall the Prometheus HA server from Workernode1.
### 
### A simple script has been developed to uninstall this Prometheus HA server. In this script, 
#### It first stop the Prometheus service.
#### Disable the Prometheus service
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the Prometheus user.
```
cat <<EOF>> /data/uninstall-prometheus-HA.sh
systemctl stop prometheus.service
systemctl disable prometheus.service
rm -rf /etc/systemd/system/prometheus.service
rm -rf /etc/prometheus/*
rmdir /etc/prometheus
rm -rf /var/lib/prometheus/*
rmdir /var/lib/prometheus
rm -rf /usr/local/bin/promtool/*
cd /data
rm -rf /data/prometheus-2.52.0.linux-amd64.tar.gz
rm -rf /data/prometheus-2.52.0.linux-amd64/*
rmdir /data/prometheus-2.52.0.linux-amd64/
userdel prometheus
rm -rf /data/prometheus-HA.sh
EOF
```

### Giving the right permission and then execute the script.
```
chmod 755 /data/uninstall-prometheus-HA.sh
sh /data/uninstall-prometheus-HA.sh
```
### Remove this script too.
```
rm -rf /data/uninstall-prometheus-HA.sh
```




## 03. Uninstall the Alertmanager HA server from Workernode2.
### 
### A simple script has been developed to uninstall this Alertmanager HA server. In this script, 
#### It first stop the Alertmanager service.
#### Disable the Alertmanager service
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the Alertmanager user.
```
cat <<EOF>> /data/uninstall-alertmanager-HA.sh
systemctl stop alertmanager
systemctl disable alertmanager
rm -rf /etc/systemd/system/alertmanager.service
rm -rf /etc/amtool
rm -rf /var/lib/alertmanager
rm -rf /etc/alertmanager
rm -rf /usr/local/bin/alertmanager
rm -rf /usr/local/bin/amtool
rm -rf /data/alertmanager-0.27.0.linux-amd64.tar.gz
rm -rf /data/alertmanager-installation.sh
rm -rf /data/alertmanager-0.27.0.linux-amd64
userdel alertmanager
EOF
```

```
chmod 755 /data/uninstall-alertmanager-HA.sh
sh /data/uninstall-alertmanager-HA.sh
```

```
rm -rf /data/uninstall-alertmanager-HA.sh
```

### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### From 
```
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["192.168.1.32:9093", "192.168.1.33:9093"]    # added only
```

### To
```
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["192.168.1.32:9093"]    # added only
```

### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click Status > Runtime & Build Information.
### Our Alertmanager HA should not now visible.








## 04. Uninstall the Alertmanager server from Workernode1.
### 
### A simple script has been developed to uninstall this Alertmanager server. In this script, 
#### It first stop the Alertmanager service.
#### Disable the Alertmanager service
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the Alertmanager user.

```
cat <<EOF>> /data/uninstall-alertmanager.sh
systemctl stop alertmanager
systemctl disable alertmanager
rm -rf /etc/systemd/system/alertmanager.service
rm -rf /etc/amtool
rm -rf /var/lib/alertmanager
rm -rf /etc/alertmanager
rm -rf /usr/local/bin/alertmanager
rm -rf /usr/local/bin/amtool
rm -rf /data/alertmanager-0.27.0.linux-amd64.tar.gz
rm -rf /data/alertmanager-installation.sh
rm -rf /data/alertmanager-0.27.0.linux-amd64
userdel alertmanager
EOF
```

```
chmod 755 /data/uninstall-alertmanager.sh
sh /data/uninstall-alertmanager.sh
```

```
rm -rf /data/uninstall-alertmanager.sh
```


### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### From 
```
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["192.168.1.32:9093"]    # added only
```

### To
```
alerting:
  alertmanagers:
    - static_configs:
   #     - targets: ["192.168.1.32:9093"]    # added only
```

### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click Status > Runtime & Build Information.
### Our Alertmanager HA should not now visible.



## 05. How to uninstall / remove HTTP Service Discover

### On NGINX VM , uninstall the HTTP Service Discover
### 
### A simple script has been developed to uninstall this HTTP Service Discover server. In this script, 
#### It first stop the nginx.service .
#### Disable the nginx.service.
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the Alertmanager user.
```
cat <<EOF>> /data/uninstall-http-service.sh
systemctl stop nginx.service 
systemctl disable nginx.service
rm -rf /usr/share/nginx/html/healthz 
yum remove nginx -y
rm -rf /etc/nginx/conf.d/default.conf 
EOF
```

```
chmod 755 /data/uninstall-http-service.sh
sh /data/uninstall-http-service.sh
```


### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### Remove this block only.
```
scrape_configs:
  - job_name: 'my_http_sd_job'                     # Added
    metrics_path: /healthz                         # Added
    http_sd_configs:                               # Added
      - url: 'http://192.168.1.7:8080/healthz'     # Added
        refresh_interval: 1m                       # Added
```


### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click on "Status" and then select "Targets". You should not see the HTTP services anymore.

## How to remove pushgateway

### We have installed the pushgateway on workernode2 
### 
### A simple script has been developed to uninstall this pushgateway. In this script, 
#### It first stop the pushgateway.
#### Disable the pushgateway.
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the pushgateway user.
```
cat <<EOF>> /data/uninstall-pushgateway.sh
systemctl stop pushgateway
systemctl disable pushgateway
rm -rf /etc/systemd/system/pushgateway.service
rm -rf /usr/local/bin/pushgateway
rm -rf /data/pushgateway-1.9.0.linux-amd64.tar.gz
rm -rf /data/pushgateway-1.9.0.linux-amd64
userdel pushgateway
EOF
```

```
chmod 755 /data/uninstall-pushgateway.sh
sh /data/uninstall-pushgateway.sh
```

```
rm -rf /data/uninstall-pushgateway.sh
```


### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### Remove this block only.
```
  - job_name: 'Pushgateway'
    honor_labels: true
    static_configs:
    - targets: ['192.168.1.33:9091']
```


### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click on "Status" and then select "Targets". You should not see the pushgateway anymore.



## 06. Uninstall the pushgateway from workernode2.

### We have installed the pushgateway on workernode2 
### 
### A simple script has been developed to uninstall this pushgateway. In this script, 
#### It first stop the pushgateway.
#### Disable the pushgateway.
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the pushgateway user.
```
cat <<EOF>> /data/uninstall-pushgateway.sh
systemctl stop pushgateway
systemctl disable pushgateway
rm -rf /etc/systemd/system/pushgateway.service
rm -rf /usr/local/bin/pushgateway
rm -rf /data/pushgateway-1.9.0.linux-amd64.tar.gz
rm -rf /data/pushgateway-1.9.0.linux-amd64
userdel pushgateway
EOF
```

```
chmod 755 /data/uninstall-pushgateway.sh
sh /data/uninstall-pushgateway.sh
```

```
rm -rf /data/uninstall-pushgateway.sh
```


### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### Remove this block only.
```
  - job_name: 'Pushgateway'
    honor_labels: true
    static_configs:
    - targets: ['192.168.1.33:9091']
```


### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click on "Status" and then select "Targets". You should not see the pushgateway anymore.



## 07. Uninstall the NodeExporter from nodes (master and workernode1)

### We have installed the NodeExporter on workernode1
### 
### A simple script has been developed to uninstall this NodeExporter. In this script, 
#### It first stop the NodeExporter.
#### Disable the NodeExporter.
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the NodeExporter user.
```
cat <<EOF>> /data/uninstall-node_exporter.sh
systemctl stop node_exporter.service
systemctl disable node_exporter.service
rm -rf /etc/systemd/system/node_exporter.service
rm -rf /usr/local/bin/node_exporter
rm -rf /data/node_exporter-1.8.1.linux-amd64.tar.gz
rm -rf /data/node_exporter-1.8.1.linux-amd64
userdel node_exporter
EOF
```

```
chmod 755 /data/uninstall-node_exporter.sh
sh /data/uninstall-node_exporter.sh
```

```
rm -rf /data/uninstall-node_exporter.sh
```


### It's time to clean the Prometheus server configuration file.

```
vi /etc/prometheus/prometheus.yml
```

### From
```
  - job_name: "node-exporter"                                      # Added
    static_configs:                                                # Added
      - targets: ["192.168.1.31:9100", "192.168.1.32:9100"]        # Added
```

### To
```
  - job_name: "node-exporter"                  # Added
    static_configs:                            # Added
      - targets: ["192.168.1.31:9100"]         # Added
```

### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click on "Status" and then select "Targets". You should see only single "node-exporter" i.e. master one. 
### As we installed the node-exporter on master node on the same server where we installed our promehteous. In order to remove the node-exporter from master node.
Login into the master node.
```
cat <<EOF>> /data/uninstall-node_exporter.sh
systemctl stop node_exporter.service
systemctl disable node_exporter.service
rm -rf /etc/systemd/system/node_exporter.service
rm -rf /usr/local/bin/node_exporter
rm -rf /data/node_exporter-1.8.1.linux-amd64.tar.gz
rm -rf /data/node_exporter-1.8.1.linux-amd64
userdel node_exporter
EOF
```

```
chmod 755 /data/uninstall-node_exporter.sh
sh /data/uninstall-node_exporter.sh
```

```
rm -rf /data/uninstall-node_exporter.sh
rm -rf /data/node_exporter.sh
```


```
vi /etc/prometheus/prometheus.yml
```


### Remove the below block only.
```
  - job_name: "node-exporter"                  # Added
    static_configs:                            # Added
      - targets: ["192.168.1.31:9100"]         # Added
```

### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Open the Prometheus server GUI.
```
http://192.168.1.31:9090
```

### Click on "Status" and then select "Targets". This time, you must see only a single prometheus job is running. 




## 08. Uninstall the Prometheus from master node.

### We have installed the Prometheus on master
### 
### A simple script has been developed to uninstall this NodeExporter. In this script, 
#### It first stop the NodeExporter.
#### Disable the NodeExporter.
#### Remove the servie file that we created, remove its directory where the binary files and configuration files are stored.
#### Remove the .tar file that we downloaded and then remove its dirtory too.
#### In last, remove the NodeExporter user.
```
cat <<EOF>> /data/uninstall-prometheus.sh
systemctl stop prometheus.service
systemctl disable prometheus.service
rm -rf /etc/systemd/system/prometheus.service
rm -rf /etc/prometheus
rm -rf /var/lib/prometheus
rm -rf /usr/local/bin/promtool/*
cd /data
rm -rf /data/prometheus-2.52.0.linux-amd64.tar.gz
rm -rf /data/prometheus-2.52.0.linux-amd64
userdel prometheus
rm -rf /data/prometheus.sh
EOF
```
### Giving the right permission and then execute the script.
```
chmod 755 /data/uninstall-prometheus.sh
sh /data/uninstall-prometheus.sh
```
### Remove this script too.
```
rm -rf /data/uninstall-prometheus.sh
```

