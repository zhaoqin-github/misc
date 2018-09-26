# Install Barbican

**NOTE: This document assumes you already have RDO installed as your OpenStack controll plane.**

## Install Barbican RPM on your RDO controller

```bash
yum install -y openstack-barbican-api python2-barbicanclient
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

vi /etc/barbican/barbican.conf

```
[DEFAULT]
transport_url = rabbit://guest:guest@<your_rdo_controller_ip>:5672/
```

```
# Host name, for use in HATEOAS-style references
#  Note: Typically this would be the load balanced endpoint that clients would use
#  communicate back with this service.
host_href = http://<your_rdo_controller_ip>:9311
```

```
# SQLAlchemy connection string for the reference implementation
# registry server. Any valid SQLAlchemy connection string is fine.
# See: http://www.sqlalchemy.org/docs/05/reference/sqlalchemy/connections.html#sqlalchemy.create_engine
# Uncomment this for local dev, putting db in project directory:
#sql_connection = sqlite:///barbican.sqlite
# Note: For absolute addresses, use '////' slashes after 'sqlite:'
# Uncomment for a more global development environment
sql_connection = mysql+pymysql://barbican:BARBICAN_DBPASS@<your_rdo_controller_ip>/barbican
```

vi /etc/barbican/barbican-api-paste.ini

```
[composite:main]
use = egg:Paste#urlmap
/: barbican_version
/v1: barbican-api-keystone
```

```
[filter:keystone_authtoken]
auth_protocol = http
auth_host = <your_rdo_controller_ip>
auth_port = 35357
signing_dir = /var/cache/barbican
paste.filter_factory = keystonemiddleware.auth_token:filter_factory
#need ability to re-auth a token, thus admin url
identity_uri = http://<your_rdo_controller_ip>:35357
admin_tenant_name = services
admin_user = barbican
admin_password = Password
auth_version = v3.0
```

# Sync Barbican DB

```bash
su -s /bin/sh -c "barbican-manage db upgrade" barbican
```

### Configure Barbican virutal host

vi  /etc/httpd/conf.d/wsgi-barbican.conf

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

```bash
systemctl restart httpd.service
mkdir /var/cache/barbican 
chown barbican:barbican /var/cache/barbican/ 
systemctl start openstack-barbican-api.service
```

## Verify if Barbican API can work

```bash
barbican secret container list --os-identity-api-version 2.0
```

Then you will be able to call Barbican API to upload your secrets and certificates. Good luck!
