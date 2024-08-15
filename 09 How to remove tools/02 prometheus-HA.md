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
rm -rf /etc/prometheus
rm -rf /var/lib/prometheus
rm -rf /usr/local/bin/promtool
rm -rf /usr/local/bin/prometheus
cd /data
rm -rf /data/prometheus-2.52.0.linux-amd64.tar.gz
rm -rf /data/prometheus-2.52.0.linux-amd64
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
