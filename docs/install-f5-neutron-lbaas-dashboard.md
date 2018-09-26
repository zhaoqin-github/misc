# Install F5 Neutron LBaaS Dashboard in RDO control plane

**NOTE: This document assumes that you already have RDO installed as your OpenStack control plane with Horizon and Barbican enabled.**

## Download and install F5 Neutron LBaaS Dashboard dev build in your RDO control plane

```bash
wget https://github.com/zhaoqin-github/misc/raw/master/rpm/f5-neutron-lbaas-dashboard-1.0.1.dev15-1.noarch.rpm
rpm -Uvh f5-neutron-lbaas-dashboard-1.0.1.dev15-1.noarch.rpm
```

## Additional configuration (optional)

Make sure F5 LBaaS dashboard has been enabled.

```bash
ls /usr/share/openstack-dashboard/openstack_dashboard/local/enabled/_1491_project*
```

If not, please manually copy to your Horizon 'enabled' directory.

```bash
cd /usr/lib/python2.7/site-packages
cp f5_neutron_lbaas_dashboard/enabled/_1491_project* /usr/share/openstack-dashboard/openstack_dashboard/local/enabled/
```

Configure Horizon virtual host.

```bash
sed -i '/  WSGIDaemonProcess/a\  WSGIApplicationGroup %{GLOBAL}'   /etc/httpd/conf.d/15-horizon_vhost.conf
```

Configure Keystone version, if your are using Keyston v2.

```bash
sed -i "s/'identity': 3/'identity': 2.0/g" /etc/openstack-dashboard/local_settings
```

## Restart Horizon

```bash
systemctl restart httpd.service
```

Then, you can use your web browser to access Horizon. F5 Neutron LBaaS Dashboard is located at 'Project'->'Network'->'Load Balancers'. Good luck!
