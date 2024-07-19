### HTTP Service Discover
### For HTTP service, we will install the NGINX web server and then create a web page. In this web page, we will add the alert details.

```
yum install nginx -y
```
### Modify the nginx configuration file and make sure that "default_type = application/json".
```
vi /etc/nginx/nginx.conf
```
```
    include             /etc/nginx/mime.types;
    default_type        application/json;
```
### Create a new "default.conf" file and our web services will be running on port 8080.
### root location = /usr/share/nginx/html
### Will mention a new page /healthz and in this file, we will add all the details.
```
cat <<EOF>> /etc/nginx/conf.d/default.conf 
server {
    listen       8080;
    listen  [::]:80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server { 
       location /healthz {
        access_log off;
        return 200;
		}
	}
EOF
```


```
cat <<EOF>> /usr/share/nginx/html/healthz 
[
    {
        "targets": ["192.168.1.7:80"],
        "labels": {
            "__meta_datacenter": "london",
            "__meta_prometheus_job": "node"
        }
    }
]
EOF
```
### We have installed the new nginx services, thus, we have to start the nginx service.
```
systemctl start nginx.service 
```
### Enable the nginx service so that after reboot the node, our nginx service must be running automatically. 
```
systemctl enable nginx.service 
```
### If you observe some issue, then you can use below command. 
```
journalctl -xeu nginx.service
```


### In Prometheus server:
### Obiously, we have to add our newly configured nginx server details in the prometheus configuration file. 
```
cat /etc/prometheus/prometheus.yml
```
```
scrape_configs:
  - job_name: 'my_http_sd_job'                     # Added
    metrics_path: /healthz                         # Added
    http_sd_configs:                               # Added
      - url: 'http://192.168.1.7:8080/healthz'     # Added
        refresh_interval: 1m                       # Added
```
### After modification of Prometheus configuration file, we have to restart the prometheus service
```
killall -HUP Prometheus
```
###  If you observe some issue, then you can use below command. 
```
journalctl -eu prometheus | tail -n 10
```
### Or you can also restart the Prometheus service
```
systemctl restart prometheus.service 
```
### It's time to check the Prometheus GUI.
```
http://192.168.1.31:9090
```
status--> target 

Status  --> Service Discovery

Status  --> Configuration

