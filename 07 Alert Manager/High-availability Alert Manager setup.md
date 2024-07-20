
# High-availability Alert Manager setup

### We will install the 2nd instances on workernode2 (192.168.1.33).
### We have already deployed the alertmanager, thus, this time we are going to install and configure the Alert Manager through script. 
### All steps are same, however, we need to add one more option "--cluster.peer=192.168.1.32:9094" in the service file.
### Please bear in mind that Port number 9093 is used for to open the Web GUI and Port 9094 is for internal communication between Alert Managers.

```
cat <<EOF>> alertmanager-installation.sh
echo -e "\033[32m create a user 'alertmanager'\033[m"
useradd -M -r -s /bin/false alertmanager
echo -e "\033[32m Download the file\033[m"
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
echo -e "\033[32m --- \033[m"
echo -e "\033[32m Extract the .tar file.\033[m"
tar xvzf alertmanager-0.27.0.linux-amd64.tar.gz 
echo -e "\033[32m ----\033[m"
echo -e "\033[32m Go inside the directory\033[m"
cd alertmanager-0.27.0.linux-amd64/
echo -e "\033[32m Copy the binary files\033[m"
cp alertmanager amtool /usr/local/bin/
echo -e "\033[32m Changing the ownership of these binary files to alertmanager, so that our user alertmanager can execute these commands.\033[m"
chown alertmanager:alertmanager /usr/local/bin/{alertmanager,amtool}
echo -e "\033[32m Create a new directory for the alertmanager configuration\033[m"
mkdir -p /etc/alertmanager
echo -e "\033[32m Move the default yaml file into our newly created directory\033[m"
cp alertmanager.yml /etc/alertmanager/
echo -e "\033[32m Changing the ownership of this directory to alertmanager, so that our user alertmanager can create or modify the file(s) inside this directory.\033[m"
chown -R alertmanager:alertmanager /etc/alertmanager

echo -e "\033[32m Create a data directory for Alertmanager\033[m"
mkdir -p /var/lib/alertmanager
echo -e "\033[32m Changing the ownership of this file to alertmanager, so that our user alertmanager can write on this directory.\033[m"
chown alertmanager:alertmanager /var/lib/alertmanager

echo -e "\033[32m Create a data directory for amtool\033[m"
mkdir -p /etc/amtool
echo "alertmanager.url: http://localhost:9093" > /etc/amtool/config.yml

echo -e "\033[32m Create a service and add one extra option '--cluster.peer=192.168.1.32:9094', our first Alertmanager IP\033[m"
echo "[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target
[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager  --config.file /etc/alertmanager/alertmanager.yml  --storage.path /var/lib/alertmanager/ --cluster.peer=192.168.1.32:9094
[Install]
WantedBy=multi-user.target" >  /etc/systemd/system/alertmanager.service


echo -e "\033[32m Start and enable the alertmanager service\033[m"
systemctl enable alertmanager
systemctl start alertmanager

systemctl status  alertmanager --lines 1
EOF
```
### Modify the permission. 
```
chmod 755 alertmanager-installation.sh
```

###  Let's execute this script.
```
sh alertmanager-installation.sh
```

### Once we validate that our AlertManager is working fine, then we need to modify our first AlertManager config also.
### For this, login into the workernode1
```
vi /etc/systemd/system/alertmanager.service
```
### Add this option "--cluster.peer=192.168.1.33:9094" at the end of line "ExecStart"
### Or we can do with the help of "sed" command.

```
cat /etc/systemd/system/alertmanager.service
```
#### -i = Insert the words in the file.
#### /ExecStart/ = It search the word "ExecStart" from the file "/etc/systemd/system/alertmanager.service".
#### s = substitute (add / delete / modify the words)
#### /$/ = at the end of line.
#### /  --cluster.peer=192.168.1.33:9094/ === It will add the string "  --cluster.peer=192.168.1.33:9094". 
#### There are some white space in the beginning. This is what we want.


```
sed -i '/ExecStart/ s/$/  --cluster.peer=192.168.1.33:9094/' /etc/systemd/system/alertmanager.service
```

```
cat /etc/systemd/system/alertmanager.service
```
### Since, We did the modification in the service file, thus, need to reoload the daemon and then followed by alertmanager service restart.

```
systemctl daemon-reload ; systemctl restart alertmanager
```

### Post checks. 

### 1. Check first Alert Manger GUI 
```
http://192.168.1.32:9093/#/alerts
```

```
http://192.168.1.33:9093/#/alerts
```

### 2. Create a Silence rule on one server and it should replicate on the 2nd Alert Manager.
## 
## 

![image](https://github.com/user-attachments/assets/cb287f3b-9b11-4f10-8cc3-93572c7f326a)

## 
## 

![image](https://github.com/user-attachments/assets/7d4d3e5c-43f5-4b70-988c-e12f8fb67117)

## 
### Same thing, you will be observed on 2nd Alertmanager.

![image](https://github.com/user-attachments/assets/c0f5391d-a091-43a3-bf05-b6490147ff61)

### From the above 2 tests, we can confirm that our Alermanagers are working in a cluster mode.


### One last thing is pending, i.e. add 2nd alertmanager IP and port number in the Prometheus configuration file.
```
vi /etc/prometheus/prometheus.yml
```
```
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["192.168.1.32:9093", "192.168.1.33:9093"]    # added only
```
### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```
### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```

### Post check!

```
http://192.168.1.31:9090
```
### Status --> Runtime & Build Information
### At the bottom of this page, you will observe your AlertManager.
