SmartOS Provision
=========

This role provisions virtual machine on a SmartOS host using Ansible.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------
```
        - hosts: shell
          gather_facts: no
          remote_user: root
          roles:
            - role: smartos_provision
          vars:
            hypervisor_ip: 10.0.3.2
            hypervisor_user: root
            ansible_python_interpreter: /usr/bin/python2.7
            autoboot: "true"
            image_uuid: c540b62c-beb2-11e5-8512-8b1694a57f84  # minimal-64-lts
            cpu_cap: 100
            max_phy_mem: 256
            quota: 5
            brand: joyent
            alias: shell 
            domain: signet
            ip: 10.0.4.10
            resolver: 10.0.4.1
            admin_group: adm
            user_script: "/usr/sbin/mdata-get root_authorized_keys > ~root/.ssh/authorized_keys ; /usr/sbin/mdata-get root_authorized_keys > ~admin/.ssh/authorized_keys"
            ssh_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCX5NmP23FhXZ+YiV3unu/Bz6h5oaeJyx3J5EaJOi4de0im3MV1aXZlpYnF0MfpmRxYl2S2pUEJXjW/toA48A+zYjHI7xReKZ9MpCsDBlW4Vfl6EjaoZqN3Hc4P5wK/BiMkSIgURFRJukus1ajRvV+YZiAaRyTwgkhmF20ZdOOIAPiugaoEYg+6iQ5CJZURw1VLJ+UViCC7cBcC4AOjKcbEaLf9RzjISzAs78fN7G60+P5fyAsIinDhKC2VJE/AkxjFtQAdBlt3HNhWnLfd2jmClRNA24Ob/gL3i3OWecWdEsERSypDiOFZI/sRHDKih1mkESbiZiHHMiZRCO34Fqpx piranha@laptop"
            nics:
              - {interface: "net0", nic_tag: "external", vlan_id: "4", ip: "10.0.4.10", netmask: "255.255.255.0", gateway: "10.0.4.1"}
              - {interface: "eth1", nic_tag: "stub0", ip: "10.0.1.3", netmask: "255.255.255.0"}
            filesystems:
              - {source: "/zones/data/somedata", target: "/media/somedata", read_only: false}
              - {source: "/zones/data/moredata", target: "/media/moredata", read_only: false}
```

Notes
-----

You must have ssh-key access to your SmartOS hypervisor in order for this to provision properly.
If your ssh-key is password protected you must use ssh-agent to decrypt it in memory:

```
ssh-agent bash
ssh-add ~/.ssh/id_rsa
ansible-playbook -i example.hosts example.yml
```

ToDo
----

- Auto-generate json for provisioning
- Allow for multi-nic support

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
