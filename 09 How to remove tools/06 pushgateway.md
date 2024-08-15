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

