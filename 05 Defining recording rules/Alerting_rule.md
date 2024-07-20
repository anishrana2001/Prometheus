## How to create a Alerting rule in Prometheus.
### Alerting rule specify the conditions that an alert should be fired to an external service 
### whereas Recording rules are for pre-calculating frequently used or computationally expensive queries.
### We can also create an Alerting rules instead of recording rules.
```
vi /etc/prometheus/rules/node_exporter.yaml
```
```
groups:
 - name: node_exporter                # Name of the group
   interval: 15s
   rules:
     - alert: NodeExporterServiceDown         # Name of the alert
       expr: up{job="node-exporter"} == 0     # == 0 means node_exporter service is down and 1 means up.
       labels:                                # We can also set the labels and we 
         severity: warning
       annotations:
         summary: Server is down
```

### Check the configuration syntax.
```
promtool check config /etc/prometheus/prometheus.yml
```
### If all are ok then, restart the Prometheus server or killHUB
```
killall -HUP prometheus
```

or 
```
systemctl restart prometheus
```

### Post check 
```
http://192.168.1.31:9090/
```

### Check the alerts. Also check the Alertmanager (if you have configured).
