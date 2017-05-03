# etcd

Ansible role for etcd deployment.


## Inventory file demo

```
frank6866-etcd-1 ansible_ssh_host=192.168.168.201 ansible_ssh_port=3322 ansible_ssh_user=centos etcd_public_ip=192.168.168.201
frank6866-etcd-2 ansible_ssh_host=192.168.168.202 ansible_ssh_port=3322 ansible_ssh_user=centos etcd_public_ip=192.168.168.202
frank6866-etcd-3 ansible_ssh_host=192.168.168.203 ansible_ssh_port=3322 ansible_ssh_user=centos etcd_public_ip=192.168.168.203

[cluster1]
frank6866-etcd-[1:3]

[etcd:children]
cluster1
```

Notice:

* etcd_public_ip variable in the inventory file is requried for etcd releated configurations
* We use the group etcd, the group "cluster1" is temporary, you can name it as you want



## Role Variables
No required vars, the optional var as follows:

If you want to set the data dir, set **etcd_data_dir** variable:


## TLS config
If you want to set TLS for etcd client-to-server and peer-to-peer communication, set the following group variables in inventory file:

```
[etcd:vars]
etcd_tls_enabled='true'
etcd_protocol=https
etcd_cert_file_path=/tmp/process_csr/etcd.crt
etcd_key_file_path=/tmp/process_csr/etcd.key
etcd_cacert_file_path=/usr/local/etc/openssl/myCA/cacert.pem
```

If you want to know how to set up our own CA and sign for cert, please refer this link: [https://frank6866.gitbooks.io/linux/content/chapters/security/security-ca-setup.html](https://frank6866.gitbooks.io/linux/content/chapters/security/security-ca-setup.html)

If you want to know how to generate the csr files, please refer this link: [https://frank6866.gitbooks.io/linux/content/chapters/db/db-etcd-security.html](https://frank6866.gitbooks.io/linux/content/chapters/db/db-etcd-security.html)


## Example Playbook

```
---
- hosts: etcd
  become: true
  roles:
    - /path/to/etcd/role
```

## Verfiy
After ansible role runs completely, you can use the following command to verfiy cluster status:

If you use https, please source the following file first:

```
# source /root/etcd_rc_files/etcd.rc
```

Then check the status of the cluster, example output as follows:

```
# etcdctl cluster-health
member 80b09b5073f8bb58 is healthy: got healthy result from https://192.168.168.202:2379
member 83bf297f97d95dfe is healthy: got healthy result from https://192.168.168.201:2379
member f7cfe6dc3f0c089b is healthy: got healthy result from https://192.168.168.203:2379
cluster is healthy
```


License
-------

MIT

