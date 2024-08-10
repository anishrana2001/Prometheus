## Installation of Alertmanager:

### We will going to install the Alertmanager on Workernode1 (192.168.1.32)
### Create a user and group for Alertmanager:
```
useradd -M -r -s /bin/false alertmanager
```

###  URL: https://github.com/prometheus/alertmanager/releases/ 

### Dowload the file on workernode1.
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```
### Extract the file.
```
tar xvzf alertmanager-0.27.0.linux-amd64.tar.gz 
```
### Go inside this directory.
```
cd alertmanager-0.27.0.linux-amd64/
```
### You will observe 2 binary files, i.e. amtool and alertmanager. Copy these files to "/usr/local/bin/" directory. So that we can execute these commands without specifying the absolute path.
```
cp alertmanager amtool /usr/local/bin/
```
### Change the owernership of both binary files to "alertmanager".
```
chown alertmanager:alertmanager /usr/local/bin/{alertmanager,amtool}
```
### Create a new directory "/etc/alertmanager"
```
mkdir -p /etc/alertmanager
```
### Copy the "alertmanager.yml" into newly created directory "/etc/alertmanager".
```
cp alertmanager.yml /etc/alertmanager/
```
### After that change the ownership to alertmanager user.
```
chown -R alertmanager:alertmanager /etc/alertmanager
```

### Create a data directory for Alertmanager:

```
mkdir -p /var/lib/alertmanager
```
### After that change the ownership to alertmanager user.
```
chown alertmanager:alertmanager /var/lib/alertmanager
```
### Create a new directory for amtool. In this directory, we will create a new config.yaml file. In this file, we will specify this VM IP or localhost variable.
```
mkdir -p /etc/amtool
```

```
cat <<EOF>> /etc/amtool/config.yml
alertmanager.url: http://localhost:9093
EOF
```
### Now, we have created all the desired directory and files. Let's create a service. 

```
cat <<EOF>> /etc/systemd/system/alertmanager.service
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target
[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager  --config.file /etc/alertmanager/alertmanager.yml  --storage.path /var/lib/alertmanager/
[Install]
WantedBy=multi-user.target
EOF
```

### Start and enable the alertmanager service:

```
systemctl enable alertmanager
```

```
systemctl start alertmanager
```

### How we can verify it ?
```
systemctl status alertmanager
```
### Verify the service is running and you can reach it:

```
curl localhost:9093
```

### Verify web portal
```
http://192.168.1.32:9093/#/alerts
```


### Verify amtool is able to connect to Alertmanager and retrieve the current configuration:
```
amtool config show
```

### Now, its time to configure the Prometheus server:

```
vi /etc/prometheus/prometheus.yml
```

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

```
http://192.168.1.31:9090
```
### Click Status > Runtime & Build Information.

```
https://prometheus.io/docs/alerting/latest/configuration/
```

## How to edit the Alertmanager config file?
### Login to Alermanager CLI

### Edit the Alertmanager configuration file

```
sudo vi /etc/alertmanager/alertmanager.yml
```

### Set global.resolve_timeout to 10m:
```
global:
 ...
 resolve_timeout: 10m
```
```
global:                       ## Added
  resolve_timeout: 10m        ## Added
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

### ResolveTimeout is the default value used by alertmanager if the alert does not include EndsAt, after this time passes it can declare the alert as resolved if it has not been updated.
### This has no impact on alerts from Prometheus, as they always include EndsAt.

### Restart Alertmanager to load the new configuration:
```
systemctl restart alertmanager
```

### Note: If you wish, you can load the new configuration without restarting Alertmanager
```
sudo killall -HUP alertmanager
```
