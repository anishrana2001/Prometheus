# File  Service Discover (File_sd)

File service Discovery is used for static content. 


### Create a Json file on master1 node. As per your requirement, you may populate the data inside the labels.

```
echo "[
  {
    "labels": {
      "job": "node"
    },
    "targets": [
      "192.168.1.32:9100"
    ]
  }
] " >  /tmp/filesd.json
```
### Add the below lines in prometheus configuration file.
```
vi /etc/prometheus/prometheus.yml
```

```
  - job_name: 'node'
    file_sd_configs:
      - files:
        - '/tmp/filesd.json'
```

### Check the syntax of file.
```
promtool check config /etc/prometheus/prometheus.yml
```


### Restart the Prometheus service.
```
systemctl restart prometheus.service
```

### Check the Prometheus GUI.
```
up
```
