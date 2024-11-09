# How to create custom metrics?

## How to Create custom metrics with the help of bash script?
## How to create a metrics from reading a file?

### How to Create custom metrics with the help of bash script?

### Let's take an example, we want to monitor one service "alertmanager", which is not being monitored by any Prometheus' exporters. Thus, we have create a metrics manually. 
### There are may ways like with the help of python or some other language you may build a custom script. 
### Since, I am Linux admin, I prefer to do by bash script. 

### Check the status of service.

```
systemctl status alertmanager.service 
```

### We can grep some key words like "Active: active"
```
systemctl status alertmanager.service | grep "Active: active (running)"
```
### Please note that whenever we execute any command and if that command executed well then it's return code always be "0".
```
echo $?
```

```
systemctl status alertmanager.service | grep "Active: active (runn)"
```

```
echo $?
```

### Let's include, if and else statement. If return code is "0" then it should print "alertmanager.service 1" else it print "alertmanager.service 0". In Prometheus "up" metric, we get to know that 1  means up and 0 means down.
```
systemctl status alertmanager.service | grep "Active: active (running)" ; if [ $? -eq 0 ];then echo "alertmanager.service 1" ; else echo "alertmanager.service 0" ; fi 
```
### One can observe that we are also getting command output and if & else output. Let's suppress the command output with the help of "q" option in grep command.
### -q, --quiet, --silent
###      Quiet; do not write anything to standard output.  Exit immediately with zero status if any
###      match is found, even if an error was detected.  Also see the -s or --no-messages option.

```
systemctl status alertmanager.service | grep -q "Active: active (running)" ; if [ $? -eq 0 ];then echo "alertmanager_service 1" ; else echo "alertmanager.service 0" ; fi 
```

```
systemctl status alertmanager.service | grep "Active: active (running)"  > /dev/null; if [ $? -eq 0 ];then echo "alertmanager_service 1" ; else echo "alertmanager.service 0" ; fi 
```
### Let's stop the service.

```
systemctl stop alertmanager.service 
```

```
systemctl status alertmanager.service | grep "Active: active (running)" ; if [ $? -eq 0 ];then echo "alertmanager.service 1" ; else echo "alertmanager.service 0" ; fi
```


```
systemctl status alertmanager.service | grep "Active: active (running)"  > /dev/null; if [ $? -eq 0 ];then echo "alertmanager_service 1" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/alertmanager/ ; else echo "alertmanager_service 0" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/alertmanager/ ; fi
```

### Post checks!

### Open the Pushgateway GUI and you should see the new variable.
```
http://192.168.1.33:9091/
```

### Open the Prometheus GUI.
```
http://192.168.1.31:9090/
```
### Search "alertmanager_service'
```
alertmanager_service
```

### Now, start the "alertmanager.service" service & check re-run the script.


```
systemctl start alertmanager.service 
```

```
systemctl status alertmanager.service | grep "Active: active (running)"  > /dev/null; if [ $? -eq 0 ];then echo "alertmanager_service 1" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/alertmanager/ ; else echo "alertmanager_service 0" | curl --data-binary @- http://192.168.1.33:9091/metrics/job/alertmanager/ ; fi
```
### Post checks!

### Open the Pushgateway GUI and you should see the new variable.
```
http://192.168.1.33:9091/
```


### Open the Prometheus GUI.
```
http://192.168.1.31:9090/
```
### Search "alertmanager_service'
```
alertmanager_service
```
### You will observe the new value of this metrics.


## How to create a metrics from reading a file?
```
cd /data/
```

```
echo "1" >  file1.txt
```

```
cat file1.txt 
```

```
backup_value=`cat file1.txt`
```

```
echo $backup_value
```

```
backup_value=`cat file1.txt` ; if [ $backup_value -eq 0 ];then echo "Backup_status 1" ; else echo "Backup_status 0" ; fi
```

```
echo "0" >  file1.txt 
```

```
backup_value=`cat file1.txt` ; if [ $backup_value -eq 0 ];then echo "Backup_status 1" ; else echo "Backup_status 0" ; fi
```
