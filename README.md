ansible-role-wireguard
======================

Ansible role that configures wireguard-based VPN network with a single gateway and multiple clients.

All communication between clients and traffic from clients to external networks goes via wireguard through the specified gateway. 


```

                          /-- wg tunnel ---> client1
                         /
Internet <---> Gateway <|---- wg tunnel ---> client2
                         \
                          \-- wg tunnel ---> client3


```

This role keeps no state on controller, and only uses the /etc/wireguard/_interface_.conf files on remotes.

Quick start
-----------

Assume you have 3 machines that have single interface `eth0` that can already ping each other:
* 192.168.0.1/24 that will be used as gateway
* 192.168.0.2/24
* 192.168.0.3/24

Create inventory file:
```
gateway ansible_host=192.168.0.1 wireguard_address=10.0.0.1/24 wireguard_forward_interface=eth0 wireguard_connect_interface=eth0
client1 ansible_host=192.168.0.2 wireguard_address=10.0.0.2/24
client2 ansible_host=192.168.0.3 wireguard_address=10.0.0.3/24

[wireguard_gateway]
gateway

[wireguard_clients]
client1
client2
```

Create playbook:
```
---
# Gateway must have a valid configuration before we configure clients
- hosts: wireguard_gateway
  tasks:
    - include_role: 
        name: ansible-role-wireguard

- hosts: wireguard_clients
  tasks:
    - include_role: 
        name: ansible-role-wireguard
```

Run playbook:

```
ansible-playbook -b -i inventory playbook.yml
```

Test by running on client1:

```
ping 10.0.0.3
```


Detailed usage
-----

Required groups:
* `wireguard_gateway` - this group should only contain a single host that will be used as gateway
* `wireguard_clients` - this groups contains clients

Gateway must be configured first. This can be achieved either by splitting the playbook in two parts like in Quick Start, or by running single playbook twice using `--limit` option. Tags may be supported in the future.

Supported gateway vars:

| Variable | Required | Default value | Description |
|----------|----------|---------------|-------------|
| `wireguard_address` | Required | N/A |Address that will be used for wireguard interface, for example: `10.0.0.1/24`|
| `wireguard_forward_interface` | Required | N/A | Interface that traffic from clients to external networks will be forwarded to |
| `wireguard_connect_interface` | Required | N/A | Gateway interface that clients will use to connect to gateway |
| `wireguard_port` | Optional | `52622` | Port that wireguard should bind to on this machine |
| `wireguard_interface` | Optional | `wg0` | Name of interface that should be created by wireguard |
| `wireguard_gateway_postup` | Optional | `iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o {{ hostvars[inventory_hostname].wireguard_forward_interface }} -j MASQUERADE` | Command that should be run after starting wireguard. Default command sets up NAT that allows clients to access external Internet using Gateway's `wireguard_forward_interface`. If this command is changed, `wireguard_forward_interface` is no longer required. |
| `wireguard_gateway_postdown` | Optional | `iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o {{ hostvars[inventory_hostname].wireguard_forward_interface }} -j MASQUERADE` | Command that should be run after stopping wireguard. Default command reverts the default `wireguard_gateway_postup`. |
| `wireguard_conf_dir` | Optional | `/etc/wireguard` | Changing not supported |

Supported clients vars:

| Variable | Required | Default value | Description |
|----------|----------|---------------|-------------|
| `wireguard_address` | Required | N/A |Address that will be used for wireguard interface, for example: `10.0.0.1/24`|
| `wireguard_port` | Optional | `52622` | Port that wireguard should bind to on this machine |
| `wireguard_interface` | Optional | `wg0` | Name of interface that should be created by wireguard
| `wireguard_conf_dir` | Optional | `/etc/wireguard` | Changing not supported |

Known issues
------------

Feel free to drop a PR that fixes those:

* Clients add themselves to gateway configuration using `blockinfile`. As a result, there's no mechanism for removing clients that are no longer in use.
* Forcing specific group names in inventory feels wrong.
* Only tested on Debian Stretch in Vagrant.
