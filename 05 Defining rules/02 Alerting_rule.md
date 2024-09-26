# How to create a Alerting rule in Prometheus.
### Alerting rule specify the conditions that an alert should be fired to an external service. (It generate the alerts)
### whereas Recording rules are for pre-calculating frequently used or computationally expensive queries. (It create a new metrix rule).
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

### Login into workernode1 (192.168.1.32) and stop the node_manager service.
```
systemctl stop node_exporter.service
```
### Verify the status of service.
```
systemctl status node_exporter.service
```
### Now, check the prometheus GUI --> Alert section

![image](https://github.com/user-attachments/assets/2011acf4-8b61-450a-912c-aafab6f35b09)


## If this is the case for you, it means that you have done this lab sucessfully. 

### Let's start the node_exporter service on Workernode1. 
```
systemctl start node_exporter.service
```
### Verify the status of service.
```

![image](https://github.com/user-attachments/assets/11233a66-141c-44c6-b822-54ddb8018834)
systemctl status node_exporter.service
```
### Refresh the Prometheus GUI page. You should observe 0 active alerts.
