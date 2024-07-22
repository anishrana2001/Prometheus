### In prometheus server under rule directory.

```
cat <<EOF>> /etc/prometheus/rules/test-alerts.yml 
groups:
- name: test-alerts
  rules:
  - alert: server one down
    expr: 1
    labels:
      severity: warning
      service: node-exporter
    annotations:
      summary: Test alerts one down
  - alert: server two down
    expr: 1
    labels:
      severity: warning
      service: node-exporter
    annotations:
      summary: Test alerts two server down
  - alert: both server down
    expr: 1
    labels:
      severity: critical
      service: node-exporter
    annotations:
      summary: Node-Exporter service is down on workernode1 and nginx servers.
EOF
```
### Check the Prometheus config Syntax check.	  
```
promtool check config /etc/prometheus/prometheus.yml
```
### Restart the service.
```
systemctl restart prometheus.service 
```
### Post check
### Check the Prometheus GUI.

```
http://192.168.1.31:9090/alerts?search=
```


### Check the Alert Manager.
```
http://192.168.1.32:9093/#/alerts
```


### In Alert Manager
```
vi /etc/alertmanager/alertmanager.yml
```
```
route:
  group_by: ['cluster1']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'webhook'
receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
```



### Restart the Alert Manager service. 

```
systemctl restart alertmanager
```

### Check the Alert Manager service.
```
systemctl status alertmanager
```

### If observe some issue, then one can run below command
```
journalctl -xu alertmanager
```



### Grouping:
```
cat > /etc/alertmanager/alertmanager.yml 
cat <<EOF>> /etc/alertmanager/alertmanager.yml 
route:
  group_by: ['cluster1']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'webhook'
  routes:
  - receiver: 'webhook'
    group_by: ['service']
    match_re:
         alertname: 'server.*down'
receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
EOF
```


```
systemctl restart alertmanager

```
```
killall -HUP alertmanager 
```
```
systemctl status alertmanager 
```

## inhibit_rules --> Supress

```
cat > /etc/alertmanager/alertmanager.yml 
cat <<EOF>> /etc/alertmanager/alertmanager.yml 
route:
  group_by: ['cluster1']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'webhook'
  routes:
  - receiver: 'webhook'
    group_by: ['service']
    match_re:
         alertname: 'server.*down'
receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'critical'
    equal: ['alertname', 'dev', 'instance']
  - source_match_re:
      alertname: 'Server.*Down'
    target_match:
      service: downstream
EOF
```
