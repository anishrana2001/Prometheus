# Recording rule

## Will going to create the recording rules on Prometheus server (master1.example.com)

### Create a directory for rules
```
mkdir -p /etc/prometheus/rules
```
### Go into this directory.
```
cd /etc/prometheus/rules
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
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

rule_files:
  - "/etc/prometheus/rules/*"      ## Added
```
### Before reloading the Prometheus service, it would be worth to check the syntax error of our Prometheus configuration file.
```
promtool check config /etc/prometheus/prometheus.yml
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


## For Alert:

```
cat > /etc/prometheus/rules/alert.yaml
```
```
groups:
 - name: nginx_server
   interval: 15s
   rules:
     - alert: node_manager_cpu_5m    # The name of the alert. Must be a valid label value.
       expr: sum((rate(node_cpu_seconds_total{job="node-exporter",mode="user",instance="192.168.1.31:9100"}[5m]))) > 0
       for: 10m                      # Alerts are considered firing once they have been returned for this long. Alerts which have not yet fired for long enough are considered pending. default = 0s
       keep_firing_for: 0s           # How long an alert will continue firing after the condition that triggered it has cleared. default = 0s
       labels:                       # Labels to add or overwrite for each alert.
          severity: critical
       annotations:                  # Annotations to add to each alert.
          description: node_manager
     - alert: HostOutOfMemory
       expr: node_memory_MemAvailable_bytes{instance="192.168.1.31:9100"} / node_memory_MemTotal_bytes{instance="192.168.1.31:9100"}  * 100 > 10
       for: 2m
       labels:
         severity: warning
       annotations:
         summary: "Host out of memory instance 192.168.1.31 : sevirity=warning"
         description: Node memory is filling up 
     - alert: HostMemoryUnderMemoryPressure
       expr: rate(node_vmstat_pgmajfault[1m]) > 1000
       for: 2m
       labels:
          severity: warning
       annotations:
          summary: Host memory under memory pressure (instance {{ $labels.instance }})
          description: "The node is under heavy memory pressure. High rate of major page faults\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### Before reloading the Prometheus service, it would be worth to check the syntax error of our Prometheus configuration file.
```
promtool check config /etc/prometheus/prometheus.yml
```

### Reload the Prometheus service
```
killall -HUP prometheus
```
### Go to Prometheus GUI.
```
http://192.168.1.31:9090/
```

### Click on "Alerts", you should see one alert.
