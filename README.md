ansible-pacemaker
=================

This is an Ansible module to configure pacemaker with crm command.  To
use this, write a playbook like below:

```
- name: test
  hosts: controller
  sudo: yes
  serial: 1
  tasks:
    - name: disable stonith
      pacemaker: >
         resource='property no-quorum-policy="ignore" stonith-enabled="false"'
         state=present

    - name: define floating IP
      pacemaker: 
         resource: >
           primitive test_vip ocf:heartbeat:IPaddr2
           params ip="192.168.33.200" cidr_netmask="24" nic="port-ctl"
         state: present

    - name: change floating IP
      pacemaker: 
         resource: >
           primitive test_vip ocf:heartbeat:IPaddr2
           params ip="192.168.33.100" cidr_netmask="24" nic="port-ctl"
         state: present

    - name: remove floating IP
      pacemaker: 
         resource: >
           primitive test_vip ocf:heartbeat:IPaddr2
           params ip="192.168.33.100" cidr_netmask="24" nic="port-ctl"
         state: absent
```
"resource" contains the crm resource to configure. 
'primitive ... nic="port-ctl"' is just like "crm configure primitive"
subcommand. As such after every call of the pacemaker module there is an implicit commit.

state=<present|absent> works just like other ansible modules (default=present).
If there is the same configuration, a task with state=present will do nothing
and one with state=absent will delete the configuration.

|current                           |state=present            |state=absent |
|----------------------------------|-------------------------|-------------|
|the same config                   |doing nothing            |delete one   |
|another config	with same id/name  |delete it and add new one|delete one   |
|no config                         |add new one              |doing nothing|

Currently, it supports crm configure sub commands below:

- primitive (tested)
- monitor
- group
- clone
- ms
- rsc_template
- location
- colocation
- order
- property (tested)
- rsc_defaults
- fencing_topology
