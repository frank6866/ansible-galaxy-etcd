etcd
==============

ansible etcd deployment role.


Inventory file demo
-------------------

```
k8s-1 ansible_ssh_host=10.10.10.5 ansible_ssh_port=3322 ansible_ssh_user=centos
k8s-2 ansible_ssh_host=10.10.10.6 ansible_ssh_port=3322 ansible_ssh_user=centos
k8s-3 ansible_ssh_host=10.10.10.7 ansible_ssh_port=3322 ansible_ssh_user=centos

[cluster1]
k8s-[1:3]

[etcd:children]
cluster1
```

Notice:

* ansible_ssh_host variable in the inventory file is requried for etcd configurations
* We use the group etcd, the group "cluster1" is temporary, you can name it as you want


Role Variables
--------------
No required vars, the optional var as follows:

If you want to set the data dir, set **ETCD_DATA_DIR** variable:

> ETCD_DATA_DIR: /var/lib/etcd/default.etcd


Example Playbook
----------------

```
---
- hosts: etcd
  become: true
  roles:
    - /path/to/etcd/role
```


License
-------

MIT

