ansible-pacemaker
=================

This is an Ansible module to configure pacemaker with crm command.  To
use this, write a playbook like below:

```
- name: test
  hosts: controller
  become: yes
  serial: 1
  ignore_target_role: false
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

To modify more than one resource in a transaction and commit all changes at once, use the "shadow"
property in combination with the actions 'prepare', 'resource' and 'commit'. This will create a shadow
copy of the running config, modify it and commit it. This new feature replaces the non-functional
commit handling of all prevoius releases.

Transactional example:

```
- name: test
  hosts: controller
  become: yes
  serial: 1
  tasks:
    - name: Start pacemaker config transation
      pacemaker:
        action: prepare
        shadow: my-temp-config
    - name: Modify resource
      pacemaker:
        resource: [...]
        shadow: my-temp-config
    - name: Modify resource
      pacemaker:
        resource: [...]
        shadow: my-temp-config
    [...]
    - name: Commit pacemaker config transation
      pacemaker:
        action: commit
        shadow: my-temp-config
```

"resource" contains the crm resource to configure. 
'primitive ... nic="port-ctl"' is just like "crm configure primitive"
subcommand. As such after every call of the pacemaker module there is an implicit commit.

if `ignore_target_role` is given and true the meta.target-role value will
be ignored when comparing current and new cib, allowing for resource restart
outside this module.

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
- group (tested)
- clone
- ms
- rsc_template
- location
- colocation
- order
- property (tested)
- rsc_defaults
- fencing_topology

compatibility
-------------

This version has been tested with crmsh 2.2.x and 2.3.x.
