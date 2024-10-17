## To set up password authentication for your SMTP server (using Postfix) so that AlertManager can send emails securely, follow these steps:
### Step 1: Update Your System
### First, ensure your system is up to date:

```
dnf update -y
```
### How to setup the hostname?
```
hostnamectl
```

```
hostnamectl set-hostname workernode1
```

```
hostnamectl
```

### How to setup FQDN in Linux?
```
vi /etc/hosts
```
```
192.168.1.8  workernode1.example.com workernode1
```
### Configure domain name in Red Hat RHEL, Fedora and CentOS
```
vi /etc/sysconfig/network
```
```
DOMAINNAME=example.com
```
```
vi /etc/sysctl.conf
```

```
kernel.domainname = example.com
```

```
dnsdomainname
```

### Create a User and Set the Password. We will create 2 users, first user "alertmanager-smtpuser" will send the alerts through email and 2nd user "smtpuser" will receive the email notifications.

```
adduser alertmanager-smtpuser
```
```
echo "Redhat@123" | passwd "alertmanager-smtpuser" --stdin
```

#### Let's create another user.
```
adduser smtpuser
```
```
echo "Redhat123" | passwd "smtpuser" --stdin
```


## Step 2: Install Postfix & Dovecot

#### Install the Postfix mail server:
```
dnf install postfix cyrus-sasl-lib libsasl2* -y
```
#### Start and enable Postfix to run on boot:
```
systemctl start postfix
systemctl enable postfix
```
#### Check the status of Postfix to ensure it’s running:
```
systemctl status postfix
```

### Installation of Dovecot
#### Install Dovecot, which will handle the IMAP and POP3 services:
```
dnf install dovecot -y
```
#### Start and enable Dovecot:
```
systemctl start dovecot
systemctl enable dovecot
```
#### Check the status of Dovecot:
```
systemctl status dovecot
```

#### Post checks for Postfixs. Ideally, it should not work as I haven't added curtail variables.
```
nc -vz 192.168.1.32 25
```

### Let's configure the Postfix configuration file. Firstly, take the backup of main configuration file. 

```
mv /etc/postfix/main.cf /etc/postfix/main.cf.bak1
```

#### Add the node name in the hostname variable so that we can use in the config file.
```
hostname=$(hostname -f)
```
#### Let's create a new Postfix configuration file.
```
sudo tee -a /etc/postfix/main.cf > /dev/null <<EOF
myhostname = workernode1.example.com
mydomain = example.com
myorigin = \$mydomain
inet_interfaces = all
inet_protocols = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 192.168.1.0/24, 127.0.0.0/8
home_mailbox = Maildir/
smtpd_banner = $myhostname ESMTP 
EOF
```
#### Resart the POSTFIX service.
```
systemctl restart postfix
```
#### It's time for POST CHECK:
```
nc -vz 192.168.1.32 25
```

## Step 3: Enable the the TLS/SSL?
### A. Create a self sign certificates.

### Create a directory first and assign the right permission.
```
mkdir /etc/ssl/private/
chmod 700 /etc/ssl/private/
cd /etc/ssl/private/
```

#### Create the CA key from openssl

```
openssl genrsa -out /etc/ssl/private/ca.key 2048
```

```
openssl req -new -x509 -days 365 -key ca.key -subj "/C=IN/ST=NEWDELHI/L=DEL/O=example, Inc./CN=example Root CA" -out /etc/ssl/private/ca.crt
```

```
openssl req -newkey rsa:2048 -nodes -keyout /etc/ssl/private/mailserver.key  -subj "/C=IN/ST=NEWDELHI/L=DEL/O=example, Inc./CN=*.example Root CA"  -out /etc/ssl/private/server.csr
```


```
openssl x509 -req -extfile <(printf "subjectAltName=DNS:example.com,DNS:workernode1.example.com") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out /etc/ssl/certs/mailserver.crt
```

```
ls -ltr
```

#### Add these certificate and keys on Postfix main.cf file.
### Create a new variable so that we can use this in the main.cf file.
```
hostname=$(hostname -f)
```

##### It's time to modify the Postfix configuration file. Easiest way to take the backup of original file and add our content. 

```
sudo mv /etc/postfix/main.cf /etc/postfix/main.cf.bak1
```

```
sudo tee -a /etc/postfix/main.cf > /dev/null <<EOF
myhostname = workernode1.example.com
mydomain = example.com
myorigin = \$mydomain
inet_interfaces = all
inet_protocols = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 192.168.1.0/24, 127.0.0.0/8
home_mailbox = Maildir/
smtpd_banner = $myhostname ESMTP 

# Additional STARTTLS configuration settings
tls_random_source=dev:/dev/urandom

# SMTPD TLS configuration for incoming connections
smtpd_use_tls = yes
smtpd_tls_cert_file = /etc/ssl/certs/mailserver.crt
smtpd_tls_key_file = /etc/ssl/private/mailserver.key
smtpd_tls_security_level = may
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain =
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = noanonymous
#smtpd_recipient_restrictions = permit_sasl_authenticated, reject_unauth_destination
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination



# SMTP TLS configuration for outgoing connections
smtp_use_tls = yes
smtp_tls_cert_file = /etc/ssl/certs/mailserver.crt
smtp_tls_key_file = /etc/ssl/private/mailserver.key
smtp_tls_security_level = may
EOF
```
#### Uncomment the submission line, which allow us to use 587 secure port.
```
sed -i 's/\#submission/submission/' /etc/postfix/master.cf 
```

#### Make sure that you have these lines added or modified under submission of master.cf file. Please note that main.cf file different then master.cf file.

```
vi /etc/postfix/master.cf
```
```
submission inet n       -       n       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
  -o smtpd_tls_auth_only=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_client_restrictions=$mua_client_restrictions
  -o smtpd_helo_restrictions=$mua_helo_restrictions
  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```
#### Restart the Postfix service.
```
systemctl restart postfix
```


#### Postchecks for Postfixs.
```
nc -vz 192.168.1.32 587
```


### Its time to configure the dovecot. 

#### Open the dovecot main configuration file.
```
vi /etc/dovecot/dovecot.conf
```
##### line 30 : uncomment (if not use IPv6, remove [::]). For me, I am not using IPV6, thus, I removed it.
```
listen = *
```

#### We need to modify 2 changes only.
```
vi /etc/dovecot/conf.d/10-auth.conf
```
##### line 10 : uncomment and change (allow plain text auth). It should be like below:
##### line 100 : add, like below:
```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

```
vi /etc/dovecot/conf.d/10-mail.conf
```
#### line 30 : uncomment and add. It should be like below:
```
mail_location = maildir:~/Maildir
```

```
vi /etc/dovecot/conf.d/10-master.conf
```
#### line 107-109 : uncomment and add like follows
```
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
```

```
vi /etc/dovecot/conf.d/10-ssl.conf
```
#### line 8 : change (not require SSL)
```
ssl = yes
```

#### Save and close the file, then restart Dovecot:
```
systemctl restart dovecot
```

#### If you observe some issue, then you can use below command.
```
journalctl -xeu dovecot.service
```

### Make sure to enable and restart the services.
```
systemctl restart postfix ; systemctl restart dovecot 
```


### Step 4: Modify the AlertManager configuration.

```
cat > /etc/alertmanager/alertmanager.yml
```

```
global:                       ## Added
  resolve_timeout: 10m        ## Added
  smtp_require_tls: true
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
    #  receiver: 'web.hook'
  receiver: 'email-notifications'
receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'alertmanager-smtpuser@example.com'
        from: 'smtpuser@example.com'
        smarthost: 'workernode1.example.com:587'
        auth_username: 'alertmanager-smtpuser'
        auth_identity: 'alertmanager-smtpuser'
        auth_password: 'Redhat@123'
```
#### Restart the alertManager service.
```
systemctl restart alertmanager.service
```
```
journalctl -u alertmanager.service 
```

```
journalctl -u postfix.service 
```
#### Post checks for email notifications.
```
ls -ltr /home/alertmanager-smtpuser/Maildir/new/
```

```
echo "This is a test email, 1" | mail -s "Test1 Postfix" alertmanager-smtpuser@example.com
```
