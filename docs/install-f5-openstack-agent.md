# Install F5 OpenStack Agent in RDO control plane

**NOTE: This document assumes that you already have RDO installed as your OpenStack control plane with Neutron LBaaS enabled.**

## Download F5 OpenStack Agent dev build

```bash
wget https://github.com/zhaoqin-github/misc/raw/master/rpm/f5-openstack-agent-9.6.5.dev1-8.noarch.rpm
```

## Install F5 OpenStack Agent

Please follow the [official installation guide](https://clouddocs.f5.com/products/openstack/agent/v9.6/) to install F5 OpenStack Agent. During the installation, please use the dev rpm package f5-openstack-agent-9.6.5.dev1-8.noarch.rpm to replace the release rpm package f5-openstack-agent-9.6.5-1.el7.noarch.rpm.

**NOTE: There is a known issue in the official F5 OpenStack Agent installation guide. The requisite of f5-openstack-agent-9.6.5-1.el7.noarch.rpm is f5-sdk 3.0.11, not f5-sdk 2.3.3. Please download f5-sdk-3.0.11-1.el7.noarch.rpm from [here](https://github.com/F5Networks/f5-common-python/releases/download/v3.0.11/f5-sdk-3.0.11-1.el7.noarch.rpm)**

## Configure Neutron

vi /etc/neutron/neutron.conf

```
[service_auth]
auth_url=http://<your_rdo_controller_ip>:35357/v2.0
admin_user = admin
admin_tenant_name = admin
admin_password=<your_rdo_admin_password>
auth_version = 2
```

## Configure Barbican F5 Agent configuration file

vi /etc/neutron/services/f5/f5-openstack-agent.ini

```
###############################################################################
# Certificate Manager
###############################################################################
cert_manager = f5_openstack_agent.lbaasv2.drivers.bigip.barbican_cert.BarbicanCertManager
#
# Two authentication modes are supported for BarbicanCertManager:
#   keystone_v2, and keystone_v3
#
#
# Keystone v2 authentication:
#
# auth_version = v2
# os_auth_url = http://localhost:5000/v2.0
# os_username = admin
# os_password = changeme
# os_tenant_name = admin
#
#
# Keystone v3 authentication:
#
auth_version = v3
os_auth_url = http://<your_rdo_controller_ip>:35357/v3
os_username = admin
os_password = <your_rdo_admin_password>
os_user_domain_name = default
os_project_name = admin
os_project_domain_name = default
```

## Restart F5 OpenStack Agent

```bash
systemctl restart f5-openstack-agent
```
