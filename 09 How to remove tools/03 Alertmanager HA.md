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
systemctl daemon-reload
EOF
```

```
chmod 755 /data/uninstall-alertmanager-HA.sh
sh /data/uninstall-alertmanager-HA.sh
```

```
rm -rf /data/uninstall-alertmanager-HA.sh
```

### It's time to clean the Prometheus server configuration file. Login into master node where I have installed the Prometheus (192.168.1.31)

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

