## Lab for Grouping
### In prometheus server under rule directory.

```
cat <<EOF>> /etc/prometheus/rules/test-alerts1.yml 
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
killall -HUP prometheus
```
### OR
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
### We have already configured our AlertMagaer on previous lab. 


### Grouping: Configuration required on Alertmanager.
### In production environment, we may have same labels but we can narrow down our target by using match_re syntax. This option we can use for selecting the Alertname or other labels.
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
    group_by: ['service']         ## It will grouping by service label.
    match_re:                     ## 
         alertname: 'server.*down'  ## It will match AlertName. It means that 2 conditions must be fulfil. 
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

## inhibit_rules --> Supress some alarms because main alarm is already firing. Router interface is down, thus, sub interfaces will be also down. We don't requires sub-interfaces alarms if we have main interface alarm.

### In the Prometheus server, create another rule file.

```
cat <<EOF>> test-alerts2.yml
groups:
- name: test-alerts
  rules:
  - alert: Server Down upstream 1
    expr: 1
    labels:
      severity: critical
      service: app
      env: dev
    annotations:
      summary: Server Upstream1 Down
  - alert: Server Down upstream 2
    expr: 1
    labels:
      severity: critical
      service: app
      env: dev
    annotations:
      summary: Server Upstream2 Down
  - alert: Server Down Downstream 1
    expr: 1
    labels:
      severity: warning
      service: app
      env: dev
    annotations:
      summary: Server Downstream1 Down
  - alert: Server Down Downstream 2
    expr: 1
    labels:
      severity: warning
      service: app
      env: dev
    annotations:
      summary: Server Downstream2 Down
  - alert: Server Down Downstream 3
    expr: 1
    labels:
       severity: warning
       service: app
       env: dev
    annotations:
       summary: Server Downstream3 Down
EOF
```

### Now, check the AlertManager and you will observe that all alarms. Let's suppress the unwanted alarms.
```
cat > /etc/alertmanager/alertmanager.yml
```
```
global:                       ## Added
  resolve_timeout: 10m        ## Added
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 1m
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
    equal: ['env', 'service']
```
### Restart the Alertmanager Service.
```
killall -HUP alertmanager 
```
```
systemctl status alertmanager 
```
### Open the Alertmanager GUI.
