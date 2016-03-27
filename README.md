SmartOS Provision
=========

This role provisions virtual machines on a SmartOS host using Ansible.

Requirements
------------

NOTE: This role is under heavy development. It is not recommended for production systems at this time. There is currently no chance of virtual machine deletions since that functionality hasn't been included yet.

Your ssh key must be installed on the root user (/root/.ssh/authorized_keys) on your SmartOS global zone.

### Python
NOTE: Python on the SmartOS global zone is now a requirement.

While it is strongly recommended setting `hypervisor_install_python` to true (file checksumming is done before install to be safe), you can still install it manually using `pkgin in python27` after installing  bootstrap-YYYYQ#-tools.tar.gz.

https://pkgsrc.joyent.com/install-on-illumos/ has the manual instructions. Ensure you use the `bootstrap-2015Q4-tools.tar.gz` file.

Set `ansible_user` and `ansible_python_interpreter` for SmartOS global zone under your inventory file like so:

```
[smartos]
smartos.local

[smartos:vars]
ansible_user=root
ansible_python_interpreter=/opt/tools/bin/python2
```

Role Variables
--------------
Most of these can be understood in the [man vmadm](https://smartos.org/man/1m/vmadm#PROPERTIES) PROPERTIES section.
```
hypervisor_host [REQUIRED]  IP or hostname of the SmartOS global zone
hypervisor_install_python [OPTIONAL] Tells Ansible to install python on SmartOS global zone (options: true or false. default: false)
provision_mode [OPTIONAL] Whether to provision virtual machine or not (options: true or false. default: false)
autoboot [OPTIONAL] (options: "true" or "false". default: "true")

image_uuid: `imgadm avail` from SmartOS global zone has possible values.
   Listed here: https://docs.joyent.com/public-cloud/instances/infrastructure/images as well.

image_name: `imgadm avail` from SmartOS global zone will show possible values. (i.e. debian-8 or base-64-lts)

  NOTE: This should be used instead of image_uuid when the latest
     version of an image is desired. If image_uuid is also set,
     image_name will be ignored.

alias [REQUIRED] Synonymous with virtual machine hostname
cpu_cap [OPTIONAL] (default: 100)
max_phy_mem [OPTIONAL] (default: 512)
quota [OPTIONAL] (default: 10)
brand [OPTIONAL] (options: joyent or lx. default: joyent)
kernel_version [OPTIONAL] (Automatically detected based on image metadata. fallback: 3.16.0)
domain [OPTIONAL]  (default: local)

user_script [OPTIONAL]  (default script below)
 It's common to use a script such as this one to copy the root_authorized_keys variable into `root` and `admin` on the newly created virtual machine:
 "/usr/sbin/mdata-get root_authorized_keys > ~root/.ssh/authorized_keys ; /usr/sbin/mdata-get root_authorized_keys > ~admin/.ssh/authorized_keys"

root_authorized_keys [REQUIRED]
  Populate this with public ssh-key that the ansible client can connect with.

resolvers [REQUIRED] Minimum of 1 resolver.
  example:
  resolvers:
    - 8.8.8.8
    - 8.8.4.4

nics [REQUIRED]
  Each listed nic has the following options:
   - interface [REQUIRED]
   - nic_tag [REQUIRED]
   - vlan_id [OPTIONAL] (default: no vlan)
   - ip [REQUIRED]
   - netmask [REQUIRED]
   - gateway [OPTIONAL]

  example:
  nics:
    - {interface: "net0", nic_tag: "external", vlan_id: "4", ip: "10.0.4.10", netmask: "255.255.255.0", gateway: "10.0.4.1"}

filesystems [OPTIONAL]
  The filesystem source must exist on the global zone before it can be shared to the virtual machine. A `zfs create zones/data/somedata` should be ran once before adding a filesystem to a virtual machine.

  Each filesystem listed has the following variables:
  - source [REQUIRED] Mount point from global zone.
  - target [REQUIRED] Where to mount inside virtual machine.
  - read_only [REQUIRED] true or false.

  example:
  filesystems:
    - {source: "/zones/data/somedata", target: "/export/somedata", read_only: false}
```
Dependencies
------------


Example Playbook
----------------
```
- hosts: example
  gather_facts: no
  remote_user: root
  roles:
    - smartos_provision
  vars:
    hypervisor_host: smartos.local
    hypervisor_install_python: true
    provision_mode: false  # default
    autoboot: "true"  # default
    image_uuid: c540b62c-beb2-11e5-8512-8b1694a57f84
    image_name: minimal-64-lts  # this will be ignored since image_uuid is used
    cpu_cap: 100  # default
    max_phy_mem: 512  # default
    quota: 10  # default
    brand: joyent  # default
    alias: example
    domain: local  # default
    user_script: "/usr/sbin/mdata-get root_authorized_keys > ~root/.ssh/authorized_keys ; /usr/sbin/mdata-get root_authorized_keys > ~admin/.ssh/authorized_keys"
    root_authorized_keys: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCX5NmP23FhXZ+YiV3unu/Bz6h5oaeJyx3J5EaJOi4de0im3MV1aXZlpYnF0MfpmRxYl2S2pUEJXjW/toA48A+zYjHI7xReKZ9MpCsDBlW4Vfl6EjaoZqN3Hc4P5wK/BiMkSIgURFRJukus1ajRvV+YZiAaRyTwgkhmF20ZdOOIAPiugaoEYg+6iQ5CJZURw1VLJ+UViCC7cBcC4AOjKcbEaLf9RzjISzAs78fN7G60+P5fyAsIinDhKC2VJE/AkxjFtQAdBlt3HNhWnLfd2jmClRNA24Ob/gL3i3OWecWdEsERSypDiOFZI/sRHDKih1mkESbiZiHHMiZRCO34Fqpx precurse"
    resolvers:
      - 8.8.8.8
      - 8.8.4.4
    nics:
      - {interface: "net0", nic_tag: "external", vlan_id: "4", ip: "10.0.4.10", netmask: "255.255.255.0", gateway: "10.0.4.1"}
      - {interface: "net1", nic_tag: "stub0", ip: "10.0.1.3", netmask: "255.255.255.0"}
    filesystems:
      - {source: "/zones/data/somedata", target: "/export/somedata", read_only: false}
      - {source: "/zones/data/moredata", target: "/export/moredata", read_only: false}
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
Because there's currently no way of telling if the Virtual Machine is already proisioned, it's advised to keep `provision_mode` set to `false` by default and running the following when you'd like to create the VM:
`ansible-playbook -i example.hosts example.yml --extra-vars "provision_mode=true"`

Current Limitations
-------------------
- VMs cannot currently be deleted or re-provisioned. If you are overwriting a currently running machine, you must manually run `vmadm destroy UUID` from SmartOS.
- If a virtual machine has is listed `~/.ssh/known_hosts` it must be removed or else it can cause errors while trying to manage the virtual machine.
- Only Zones (joyent/lx) are tested for provisioning at this time.

License
-------

BSD

Author Information
------------------

Andrew Klaus (andrewklaus@gmail.com)
