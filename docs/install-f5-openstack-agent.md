# Install F5 OpenStack Agent in RDO control plane

**NOTE: This document assumes that you already have RDO installed as your OpenStack control plane with Neutron LBaaS enabled.**

## Download F5 OpenStack Agent dev build

```bash
wget https://github.com/zhaoqin-github/misc/raw/master/rpm/f5-agent-package.noarch.rpm
```

## Install F5 OpenStack Agent

Please follow the [official installation guide](https://clouddocs.f5.com/products/openstack/agent/v9.6/) to install F5 OpenStack Agent. During the installation, please use the dev rpm package f5-agent-package.noarch.rpm to replace the release rpm package f5-openstack-agent-9.6.5-1.el7.noarch.rpm.

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
