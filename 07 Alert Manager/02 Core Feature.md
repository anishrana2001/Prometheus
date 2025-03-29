## Lab for Grouping

### I am going to create 3 rules, on which 2 rules will show inside a single group on AlertManager.

### In prometheus server under rule directory.
```
cat <<EOF>>  /etc/prometheus/rules/grouping-rule.yml
groups:
- name: test-alerts
  rules:
  - alert: server one down
    expr: 1
    labels:
      severity: warning
      service: node-exporter
    annotations:
      summary: Server 1 Down
  - alert: server two down
    expr: 1
    labels:
      severity: warning
      service: node-exporter
    annotations:
      summary: Server 2 down
  - alert: Nginx server Down
    expr: 1
    labels:
      severity: critical
      service: node-exporter
    annotations:
      summary: Nginx Server Down.
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



### We have already configured our AlertMagaer on previous lab. 
### As of now, we haven't observed Groupping, the main reason is that we need to modify some setting on AlertManager Config file.

### Grouping: Configuration required on Alertmanager.
### In production environment, we may have same labels but we can narrow down our target by using match_re syntax. This option we can use for selecting the Alertname or other labels.
```
cat > /etc/alertmanager/alertmanager.yml
```
```
cat <<EOF>> /etc/alertmanager/alertmanager.yml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
  - receiver: 'web.hook'
    group_by: ['service']         ## It will grouping by service label.
    match_re:                     ## Match Regular Expression
         alertname: 'server.*down'  ## It will match AlertName. It means that 2 conditions must be fulfil.
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
EOF
```


### Check the AlertManager config Syntax
```
amtool check-config /etc/alertmanager/alertmanager.yml
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
### Check the AlertManager GUI for the POST CHECK.

#### Let's change the web.hok name to webhook 

```
> /etc/alertmanager/alertmanager.yml
```
```
cat <<EOF>> /etc/alertmanager/alertmanager.yml
route:
  group_by: ['cluster1']   ## Modified
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'webhook'    ## Modified
  routes:
  - receiver: 'webhook'   ## Modified
    group_by: ['service']
    match_re:
         alertname: 'server.*down'
receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
EOF
```
### Check the AlertManager config Syntax
```
amtool check-config /etc/alertmanager/alertmanager.yml
```


### Restart the Alertmanager Service.
```
killall -HUP alertmanager 
```
```
systemctl status alertmanager 
```
## inhibit_rules --> Supress some alarms because main alarm is already firing. Router interface is down, thus, sub interfaces will be also down. We don't requires sub-interfaces alarms if we have main interface alarm.

### In the Prometheus server, create another rule file.

```
cat <<EOF>> /etc/prometheus/rules/inhibit_rules.yml
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

### Check the Prometheus config syntax
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload Prometheus configuration to load the new configuration:
```
killall -HUP prometheus
```
### Post check, Open the Prometheus GUI and click on "Alert". You must observe new alarms for upstream and downstream.

### Now, check the AlertManager and you will observe that all alarms. Let's suppress the unwanted alarms.
```
cat > /etc/alertmanager/alertmanager.yml
```
```
global:                  
  resolve_timeout: 10m    
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
inhibit_rules:                                ## Added
  - source_match:                             ## Added
      severity: 'critical'                    ## Added, If these alarms observed then it will supress target alarms (severity: 'warning') which which must have 2 lables i.e. env and service.
    target_match:                             ## Added
      severity: 'warning'                     ## Added
    equal: ['env', 'service']                 ## Added
```

### Check the AlertManager config Syntax
```
amtool check-config /etc/alertmanager/alertmanager.yml
```


### Restart the Alertmanager Service.
```
killall -HUP alertmanager 
```
```
systemctl status alertmanager 
```
### Open the Alertmanager GUI.
