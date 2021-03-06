# etcd

This role is used to deploy etcd cluster. Supporting features as follows:

* etcd standalone deployment with one member
* etcd cluster deployment with multiple members
* TLS configuration on peer-to-peer and client-server(This is strongly recommended for safety)
* Putting root password in rc file and simplify the etcdctl command

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

For example, you can run the following commands on CA server(change the IP according to all the etcd_public_ip):

```
mkdir /tmp/process_csr
openssl genrsa -out /tmp/process_csr/etcd.key 2048
openssl req -new -batch -days 3650 -subj /CN=etcd-test -key /tmp/process_csr/etcd.key -reqexts SAN -config <(cat /usr/local/etc/openssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=IP:192.168.168.201,IP:192.168.168.202,IP:192.168.168.203")) -out /tmp/process_csr/etcd.csr
openssl ca -in /tmp/process_csr/etcd.csr  -extensions SAN -config <(cat /usr/local/etc/openssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=IP:192.168.168.201,IP:192.168.168.202,IP:192.168.168.203"))  -out /tmp/process_csr/etcd.crt
```

## root password
This role does not support enable auth because creating root user is interactive. But you can put root password in rc file by setting **etcd_root_password** variable:

```
etcd_root_password: "change_me"
```

After you run this role completely, ou can enable auth manually by running the following command(the password of root you set should be the same with **etcd_root_password** variable):

```
# etcdctl user add root
New password:
User root created

# etcdctl auth enable
Authentication Enabled
```

If you enable auth, you should reovke guest role permission for security:

```
# etcdctl role revoke guest --path=/* --readwrite
```

But if you use Kubernetes storing data in etcd, you should not reovke guest permission, because Kubernetes does not support etcd authentication.

## Example Playbook

```
---
- hosts: etcd
  become: true
  roles:
    - /path/to/etcd/role
```

## Verify
If you donot config TLS for etcd, you can just run the following command to check the status of etcd cluster:

```
# etcdctl cluster-health
```

### TLS Verify
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

