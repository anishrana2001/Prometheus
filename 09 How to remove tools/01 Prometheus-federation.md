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
rm -rf /etc/prometheus
rm -rf /var/lib/prometheus
rm -rf /usr/local/bin/promtool
rm -rf /usr/local/bin/prometheus
cd /data
rm -rf /data/prometheus-2.52.0.linux-amd64.tar.gz
rm -rf /data/prometheus-2.52.0.linux-amd64
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
