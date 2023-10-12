This implements our temporary workaround for connecting private networks to floating ip addresses.

## Install

```
pip3.11 install pipenv
pipenv install -r requirements.txt
pipenv run ansible-galaxy install -r requirements.yml --force
```

## Run

```
pipenv run ansible-playbook playbook.yaml
```

## Configure

This playbook expects a `project_networks` variable that is a list of dictionaries; each dictionary identifies a project name and a network name. For example:

```
project_networks:
- project: larsks
  network: openshift
- project: tzumainn
  network: tzumainn
```

We create interfaces named `pj<network_id>`, where `network_id` is truncated to 13 characters (because the max. length of an interface name is 15 characters).

Given the above example configuration, we end up with `/etc/sysconfig/network-scripts/ifcfg-larsks-openshift` that looks something like:

```
# DO NOT EDIT: This file is managed by Ansible
# project: larsks
# network: openshift
DEVICE=pj2213ecfb7ad84
ONBOOT=yes
HOTPLUG=no
NM_CONTROLLED=no
PEERDNS=no
DEVICETYPE=ovs
TYPE=OVSIntPort
OVS_BRIDGE=br-ex
OVS_OPTIONS="tag=559"
BOOTPROTO=none
MTU=1500
IPADDR=192.168.71.253
PREFIX=24
```

Which results in an interface that looks like:

```
118: pj2213ecfb7ad84: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 9e:c0:0e:7e:f6:c1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.71.253/24 brd 192.168.71.255 scope global pj2213ecfb7ad84
       valid_lft forever preferred_lft forever
    inet6 fe80::9cc0:eff:fe7e:f6c1/64 scope link
       valid_lft forever preferred_lft forever
```
