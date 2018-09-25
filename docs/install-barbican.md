# Install Barbican

This document assumes you already have RDO installed as your OpenStack controll plane.

## Install Barbican RPM on your RDO controller

```bash
yum install -y openstack-barbican-api
yum install -y python-barbicanclient
```

## Create Barbican DB

```bash
export IP=<your_rdo_controller_ip>
mysql -u root -e "CREATE DATABASE barbican;"
mysql -u root -e "GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'${IP}' IDENTIFIED BY 'BARBICAN_DBPASS';"
mysql -u root -e "GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'%' IDENTIFIED BY 'BARBICAN_DBPASS';"
```

## Create Barbican user, role and service endpoint in Keystone

```bash
export IP=<your_rdo_controller_ip>
openstack user create --project services --password Password barbican
openstack role add --project services --user barbican admin
openstack role create creator
openstack role add --project services --user barbican creator
openstack service create --name barbican --description "Key Manager" key-manager
openstack endpoint create --region RegionOne key-manager --internalurl http://${IP}:9311 --publicurl http://${IP}:9311 --adminurl http://${IP}:9311
```

## Configure Barbican

vim /etc/barbican/barbican.conf

```
[default]
transport_url = rabbit://guest:guest@localhost:5672/
sql_connection = mysql+pymysql://barbican:BARBICAN_DBPASS@localhost/barbican
```

vim /etc/barbican/barbican-api-paste.ini

```
[composite:main]
use = egg:Paste#urlmap
/: barbican_version
/v1: barbican-api-keystone

[filter:keystone_authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory
auth_host = localhost
signing_dir = /var/cache/barbican
auth_port = 35357
auth_protocol = http
admin_tenant_name = admin
admin_user = admin
admin_password = XXXXXXXXXX
```

# Sync Barbican DB

```bash
su -s /bin/sh -c "barbican-manage db upgrade" barbican
```

### Configure Barbican virutal host

vim  /etc/httpd/conf.d/wsgi-barbican.conf

```
<VirtualHost [::1]:9311> 
    ServerName controller
    ## Logging 
    ErrorLog "/var/log/httpd/barbican_wsgi_main_error_ssl.log" 
    LogLevel debug 
    ServerSignature Off 
    CustomLog "/var/log/httpd/barbican_wsgi_main_access_ssl.log" combined
    WSGIApplicationGroup %{GLOBAL} 
    WSGIDaemonProcess barbican-api display-name=barbican-api group=barbican processes=2 threads=8 user=barbican 
    WSGIProcessGroup barbican-api 
    WSGIScriptAlias / "/usr/lib/python2.7/site-packages/barbican/api/app.wsgi" 
    WSGIPassAuthorization On 
</VirtualHost>
```

## Start Barbican
systemctl restart httpd.service
mkdir /var/cache/barbican 
chown barbican:barbican /var/cache/barbican/ 
systemctl start openstack-barbican-api.service

## Verify if Barbican API can work

```bash
barbican secret container list --os-identity-api-version 2.0
```

## Config F5 LBaaS Agent

vim /etc/neutron/services/f5/f5-openstack-agent.ini

```
cert_manager = f5_openstack_agent.lbaasv2.drivers.bigip.barbican_cert.BarbicanCertManager
```

Restart F5 LBaaS Agent

```bash
systemctl restart f5-openstack-agent
```
