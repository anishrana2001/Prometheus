

### Create a directory for rules
```
mkdir -p /etc/prometheus/rules
```
### Create a rule file. Remember that we have to create the YAML file.
```
vi global_rule.yaml
```
```
groups:
 - name: nginx_server
   interval: 15s
   rules:
   - record: recording_rule_node_cpu_seconds_total_5m
     expr: (rate(node_cpu_seconds_total{job="node-exporter",mode="user",instance="192.168.1.31:9100"}[5m]))
```

### Modify the Prometheus configuration file.

```
vi /etc/prometheus/prometheus.yml
```
```
# my global config
global:
  scrape_interval: 15s 
  evaluation_interval: 15s 
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["192.168.1.32:9093"]

rule_files:                        ## Added
  - "/etc/prometheus/rules/*"      ## Added
```

### Reload the Prometheus service
```
killall -HUP prometheus
```
### Go to Prometheus GUI.
```
http://192.168.1.31:9090/
```

```
recording_rule_node_cpu_seconds_total_5m
```
