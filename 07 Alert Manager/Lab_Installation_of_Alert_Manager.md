## Installation of Alertmanager:

### Create a user and group for Alertmanager:
```
useradd -M -r -s /bin/false alertmanager
```

# URL: https://github.com/prometheus/alertmanager/releases/        ---> Go down to the page.
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```

```
tar xvzf alertmanager-0.27.0.linux-amd64.tar.gz 
```

```
cd alertmanager-0.27.0.linux-amd64/
```

```
cp alertmanager amtool /usr/local/bin/
```

```
chown alertmanager:alertmanager /usr/local/bin/{alertmanager,amtool}
```

```
mkdir -p /etc/alertmanager
```

```
cp alertmanager.yml /etc/alertmanager/
```

```
chown -R alertmanager:alertmanager /etc/alertmanager
```

### Create a data directory for Alertmanager:

```
mkdir -p /var/lib/alertmanager
```

```
chown alertmanager:alertmanager /var/lib/alertmanager
```

```
mkdir -p /etc/amtool
```

```
cat <<EOF>> /etc/amtool/config.yml
alertmanager.url: http://localhost:9093
EOF
```


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

### Verify the service is running and you can reach it:

```
systemctl status alertmanager
```

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
