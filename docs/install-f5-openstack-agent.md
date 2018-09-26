# Install F5 OpenStack Agent in RDO control plane

**NOTE: This document assumes that you already have RDO installed as your OpenStack control plane with Neutron LBaaS enabled.**

## Download and install F5 OpenStack Agent dev build

```bash
wget https://github.com/zhaoqin-github/misc/blob/master/rpm/f5-agent-package.noarch.rpm
```

Then, please follow the [official installation guide](https://clouddocs.f5.com/products/openstack/agent/v9.6/) to install F5 OpenStack Agent. During the installation, please use the dev rpm package f5-agent-package.noarch.rpm to replace the release rpm package f5-openstack-agent-9.6.5-1.el7.noarch.rpm.
