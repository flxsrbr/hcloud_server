hcloud_server
=========

Role to manage server ressources on Hetzner Cloud.

Requirements
------------

* Hetzner Python library on target host: [hcloud](https://pypi.org/project/hcloud/)
* Ansible Hetzner collection on control node: [Hetzner.Hcloud](https://docs.ansible.com/ansible/latest/collections/hetzner/hcloud/)

Role Variables
--------------

**Default Variables**

Variables for api connection

| Variable                     | Type | Description                                                        | Default                      |
|------------------------------|------|--------------------------------------------------------------------|------------------------------|
| hcloud_sshkey_api_token_rw   | str  | Read-write Hetzner Cloud api token                                 | -                            |
| hcloud_sshkey_api_token_ro   | str  | Read-only Hetzner Cloud api token, defaults to rw-token if not set | hcloud_sshkey_api_token_rw   |

**Server Instance Variables**

Variables for provisioning of server instances. Sensible minimal default values are predefined but should be overwritten during role execution (see example below).

| Variable                          | Type    | Description                                                                                      | Default            | Example              |
|-----------------------------------|---------|--------------------------------------------------------------------------------------------------|--------------------|----------------------|
| hcloud_server_state               | str     | State of server                                                                                  | present            | absent               |
| hcloud_server_name                | str     | Name of server                                                                                   | inventory_hostname | my-server-01         |
| hcloud_server_type                | str     | Hetzner Cloud instance type (e.g. *cx11*)                                                        | cx11               | cpx21                |
| hcloud_server_image               | str     | Hetzner Cloud image                                                                              | ubuntu-22.04       | rocky-9              |
| hcloud_server_backup              | bool    | Activation of automated backups                                                                  | false              | true                 |
| hcloud_server_datacenter          | str     | Hetzner Cloud datacenter                                                                         | fsn1-dc14          | nbg1-dc3             |
| hcloud_server_public_ipv4_enabled | bool    | Provision public ipv4 address? Currently at least ipv4 or ipv6 is necessary, as module does not allow to diretly attach to internal network | true | false |
| hcloud_server_public_ipv6_enabeld | bool    | Provision public ipv6 address? Currently at least ipv4 or ipv6 is necessary, as module does not allow to diretly attach to internal network | true | false |
| hcloud_server_placement_group     | str     | Name of placement group server should be in, must exist already                                  | ""                 | "placement-group-01" |
| hcloud_server_ssh_keys            | lst/str | List of public ssh keys to inject into instance, must be names of keys existing in Hetzner Cloud | []                 | - ansible            |
| hcloud_server_firewalls           | lst/str | List of Hetzner Cloud firewalls applied to server                                                | []                 | - my-firewall-01     |
| hcloud_server_volumes             | lst/str | List of Hetzner Cloud volumes attached to server                                                 | []                 | - my-volume-01     |

**Server Private Network Variables**

To attach server to internal networks, additional variable can be set:

| Variable               | Subitem | Type     | Description                                            | Default | Example       |
|------------------------|---------|----------|--------------------------------------------------------|---------|---------------|
| hcloud_server_networks |         | lst/dict | List of internal networks server should be attached to | []      |               |
|                        | name    | str      | Name of network                                        |         | my-network-01 |
|                        | ip      | str      | IP of server in internal network                       |         | 10.0.0.100    |
|                        | state   | str      | State of attachment to internal network                |         | present       |

Tags
----

Granular role execution can be set via tags:

| Tag                    | Description                                 |
|------------------------|---------------------------------------------|
| provsion               | Only run provisioning tasks                 |
| networks               | Only run networking subtasks                |
| info                   | Only get info on current networks           |

Dependencies
------------

none

Example Playbook
----------------

All server parameters can and should be defined when calling the role. For example to provision a single host:

    - name: Manage server on Hetzner Cloud.
      hosts: my-server-01
      gather_facts: false
      vars_files:
        - vars/vault.yml

      roles:

        - name: flxsrbr.hcloud_server
          vars:
            hcloud_server_api_token_rw: "{{ vault_hcloud_token_rw }}"
            hcloud_server_api_token_ro: "{{ vault_hcloud_token_ro }}"
            hcloud_server_state: present
            hcloud_server_name: "{{ inventory_hostname }}"
            hcloud_server_type: cx11
            hcloud_server_image: ubuntu-22-.4
            hcloud_server_backup: false
            hcloud_server_datacenter: fsn1-dc14
            hcloud_server_public_ipv4_enabled: true
            hcloud_server_public_ipv6_enabled: true
            hcloud_server_placement_group: my-placement-group
            hcloud_server_ssh_keys:
              - ansible
            hcloud_server_firewalls:
              - my-firewall-01
            hcloud_server_networks:
              - name: internal
                ip: 10.0.0.100
                state: present
            hcloud_server_volumes:
              - my-volume-01

Better yet: define all server parameters as inventory, group and/or host vars and reference them when calling the role:

    - name: Manage servers on Hetzner Cloud.
      hosts: hcloud-servers
      gather_facts: false
      vars_files:
        - vars/vault.yml

      roles:

        - name: flxsrbr.hcloud_server
          vars:
            hcloud_server_api_token_rw: "{{ vault_hcloud_token_rw }}"
            hcloud_server_api_token_ro: "{{ vault_hcloud_token_ro }}"
            hcloud_server_state: "{{ server_state }}"
            hcloud_server_name: "{{ inventory_hostname }}"
            hcloud_server_type: "{{ server_type }}"
            hcloud_server_image: "{{ server_image }}"
            hcloud_server_backup: "{{ server_backups }}"
            hcloud_server_datacenter: fsn1-dc14
            hcloud_server_public_ipv4_enabled: "{{ server_ip4_enabled }}"
            hcloud_server_public_ipv6_enabled: "{{ server_ip6_enabled }}"
            hcloud_server_placement_group: "{{ server_placement_group }}"
            hcloud_server_ssh_keys: "{{ server_sshkeys }}"
            hcloud_server_firewalls: "{{ server_firewalls }}"
            hcloud_server_networks: "{{ server_networks }}"
            hcloud_server_volumes: "{{ server_volumes }}"

License
-------

BSD

Author Information
------------------

[flxsrbr](https://github.com/flxsrbr)
