Ansible Role: ZeroTier
=========

[![CI](https://github.com/AjayKumar4/ansible-role-zerotier/workflows/CI/badge.svg?event=push)](https://github.com/AjayKumar4/ansible-role-zerotier/actions?query=workflow%3ACI) [![GitHub issues](https://img.shields.io/github/issues/AjayKumar4/ansible-role-zerotier.svg)](https://github.com/AjayKumar4/ansible-role-zerotier/issues)

This Ansible role adds the ZeroTier repository and installs the `zerotier` package using your system's package manager. Depending on the provided variables this role can also add and authorize new members to (existing) ZeroTier networks, and tell the new member to join the network.

If you need a more flexible and generic role for ZeroTier, check out [`ajaykumar4.zerotier`](https://galaxy.ansible.com/ajaykumar4/zerotier).

Requirements
------------

Technically this role has no requirements. If it's ran without any variables set it will only run the installation tasks. The following variables impact the role's behavior:

[**zerotier_network_id**](#zerotier_network_id): when set hosts are told to join this network.  
[**zerotier_api_accesstoken**](#zerotier_api_accesstoken): when set the role can handle member authentication and configuration using the ZeroTier API.  

Role Variables
--------------

### zerotier_network_id
*Type*: string  
*Default value*:  
*Description*: The 16 character network ID of the network the new members should join. The node will not join any network if omitted.

### zerotier_member_register_short_hostname
*Type*: boolean  
*Default value*: `false`  
*Description*: By default `inventory_hostname` will be used to name a member in a network. If set to `true`, `inventory_hostname_short` will be used instead.

### zerotier_member_ip_assignments
*Type*: list  
*Default value*: `[]`  
*Description*: A list of IP addresses to assign this member. The member will be automatically assigned an address on the network if left out.

### zerotier_member_description
*Type*: string  
*Default value*: `""`  
*Description*: Optional description for a member.

### zerotier_api_accesstoken
*Type*: string  
*Default value*: `""`  
*Description*: The access token needed to authorize with the ZeroTier API. You can generate one in your account settings at https://my.zerotier.com/. If this is left out then the newly joined member will not be automatically authorized.

### zerotier_api_url
*Type*: string  
*Default value*: `https://my.zerotier.com`  
*Description*: The url where the Zerotier API lives. Must use HTTPS protocol.  

### zerotier_api_delegate
*Type*: string  
*Default value*: `localhost`  
*Description*: Option to delegate tasks for Zerotier API calls. This is useful in a situation where API calls can only be made from a white-listed management server, for example.

## Usage
----------------

First create a new directory based on the `sample` directory within the `inventory` directory:

```bash
cp -R inventory/sample inventory/my-cluster
```

Second, edit `inventory/my-cluster/hosts.ini` to match the system information gathered above.

Example Inventory
----------------

```INI
    [servers]
    pi-master
    pi-worker01
    jetson-worker01

    [master]
    pi-master

    [nodes]
    pi-worker01
    jetson-worker01

    [master:vars]
    zerotier_member_description='Kubernetes Pi Master Node'

    [nodes:vars]
    zerotier_member_description='Kubernetes Pi Workers Node'
```
```INI
    [servers]
    pi-master hostname=pi-master ansible_host=192.168.0.82 ansible_user=pi ansible_sudo_pass=password zerotier_member_description="Kubernetes Pi Master Node"
    pi-worker01 ansible_port=22 hostname=pi-worker01 ansible_host=192.168.0.176 ansible_user=pi ansible_sudo_pass=password zerotier_member_description="Kubernetes Pi Worker01 Node"
    jetson-worker01 ansible_port=22 hostname=jetson-worker01 ansible_host=192.168.0.227 ansible_user=jetson ansible_sudo_pass=password zerotier_member_description="Kubernetes Jetson Worker01 Node"
```

Third, edit `inventory/my-cluster/site.yml` to match the zerotier information gathered above.

Example Playbook
----------------

```yaml
    - hosts: servers
      vars:
         zerotier_network_id: < network id >
         zerotier_api_accesstoken: < accesstoken >
         zerotier_register_short_hostname: true

      roles:
         - ajaykumar4.zerotier
```

Start provisioning of the cluster using the following command:

```bash
ansible-playbook inventory/my-cluster/playbook.yml -i inventory/my-cluster/hosts.ini
```