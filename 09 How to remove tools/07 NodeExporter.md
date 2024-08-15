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


