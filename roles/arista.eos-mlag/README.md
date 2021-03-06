
MLAG Role for EOS
=================

The arista.eos-mlag role creates an abstraction for common MLAG configuration.
This means that you do not need to write any ansible tasks. Simply create
an object that matches the requirements below and this role will ingest that
object and perform the necessary configuration.


Installation
------------

```
ansible-galaxy install arista.eos-mlag
```


Requirements
------------

Requires an SSH connection for connectivity to your Arista device. You can use
any of the built-in eos connection variables, or the convenience ``provider``
dictionary.


Role Variables
--------------

The ``mlag`` dictionary includes the following keys described below:

|                            Key | Type                                | Notes                                    |
| -----------------------------: | ----------------------------------- | ---------------------------------------- |
|                 mlag_domain_id | string                              | The name of the MLAG Domain              |
|               mlag_trunk_group | string                              | Trunk group assigned to Vlan             |
|                  mlag_shutdown | boolean: true, false*               | Enable or disable the MLAG configuration |
|                  local_if_vlan | string (required)                   | The Vlan for the peer link, eg Vlan1024  |
|      local_if_vlan_description | string                              | Description for local_if_vlan            |
|            local_if_ip_address | string (required)                   | IP Address for the local_if_vlan         |
| local_if_disable_spanning_tree | boolean: true*, false               | Enable or disable STP on the peer Vlan   |
|                   peer_address | string (required)                   | IP Address for the MLAG Peer             |
|                   peer_link_if | string (required)                   | The Port-Channel used for the peer link  |
|                 peer_link_mode | choices: trunk*, access             | The switchport mode for the peer link    |
|            peer_link_lacp_mode | choices: active*, passive, disabled | The LACP mode for each Port-Channel member. |
|               peer_link_enable | boolean: true*, false               | Enable or disable the peer link member interfaces |
|              peer_link_members | (List)                              | List of interfaces that make up the peer link. |
|                          state | boolean: present*, absent           | Whether to add or remove all mlag-related configuration.  When set to absent, all configuration, including local_if_vlan and peer_link_if, will be removed and the mlag configuration block will be defaulted. Interfaces in peer_link_members will be reset to default switchport settings. |

```
Note: Asterisk (*) denotes the default value if none specified
```

Configuration Variables
-----------------------

|                     Key | Choices      | Description                              |
| ----------------------: | ------------ | ---------------------------------------- |
| eos_save_running_config | true*, false | Specifies whether to write any changes to the running-config resulting from the role execution to memory, copying the configuration to the startup-config. |

```
Note: Asterisk (*) denotes the default value if none specified
```

Connection Variables
--------------------

Ansible EOS roles require the following connection information to establish
communication with the nodes in your inventory. This information can exist in
the Ansible group_vars or host_vars directories, or in the playbook itself.

|         Key | Required | Choices    | Description                              |
| ----------: | -------- | ---------- | ---------------------------------------- |
|        host | yes      |            | Specifies the DNS host name or address for connecting to the remote device over the specified *transport*. The value of *host* is used as the destination address for the transport. |
|        port | no       |            | Specifies the port to use when building the connection to the remote device. This value applies to either acceptable value of *transport*. The port value will default to the appropriate transport common port if none is provided in the task (cli=22, http=80, https=443). |
|    username | no       |            | Configures the usename to use to authenticate the connection to the remote device.  The value of *username* is used to authenticate either the CLI login or the eAPI authentication depending on which *transport* is used. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_USERNAME will be used instead. |
|    password | no       |            | Specifies the password to use to authenticate the connection to the remote device. This is a common argument used for either acceptable value of *transport*. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_PASSWORD will be used instead. |
| ssh_keyfile | no       |            | Specifies the SSH keyfile to use to authenticate the connection to the remote device. This argument is only used when *transport=cli*. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_SSH_KEYFILE will be used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter priviledged mode on the remote device before sending any commands. If not specified, the device will attempt to excecute all commands in non-priviledged mode. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_AUTHORIZE will be used instead. |
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device.  If *authorize=no*, then this argument does nothing. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_AUTH_PASS will be used instead. |
|   transport | yes      | cli*, eapi | Configures the transport connection to use when connecting to the remote device. The *transport* argument supports connectivity to the device over cli (ssh) or eapi. |
|     use_ssl | no       | yes*, no   | Configures the transport to use SSL if set to true only when *transport=eapi*.  If *transport=cli*, this value is ignored. |
|    provider | no       |            | Convience method that allows all the above connection arguments to be passed as a dict object. All constraints (required, choices, etc) must be met either by individual arguments or values in this dict. |

```
Note: Asterisk (*) denotes the default value if none specified
```


Ansible Variables
-----------------

|    Key | Choices      | Description                              |
| -----: | ------------ | ---------------------------------------- |
| no_log | true, false* | Prevents module arguments and output from being logged during the playbook execution. By default, no_log is set to true for tasks that gather and save EOS configuration information to reduce output size. Set to true to prevent all output other than task results. |

```
Note: Asterisk (*) denotes the default value if none specified
```


Dependencies
------------

The eos-bridging role is built on modules included in the core Ansible code.
These modules were added in ansible version 2.1

- Ansible 2.1.0


Example Playbook
----------------

The following example will use the arista.eos-mlag role to completely setup MLAG
on two leaf switches without writing any tasks. We'll create a ``hosts`` file
with our two leaf switches, then a corresponding ``host_vars`` file for each
leaf and then a simple playbook which only references the mlag role. By including
the role we automatically get access to all of the tasks to configure MLAG. What's
nice about this is that if you have a host without MLAG configuration, the
tasks will be skipped without any issue.


Sample hosts file:

    [leafs]
    leaf1.example.com
    leaf2.example.com

Sample host_vars/leaf1.example.com

    provider:
      host: "{{ inventory_hostname }}"
      username: admin
      password: admin
      use_ssl: no
      authorize: yes
      transport: cli
    
    mlag:
      mlag_domain_id: mlag1
      mlag_trunk_group: mlagpeer
      mlag_shutdown: false
      local_if_vlan: Vlan1024
      local_if_vlan_description: Peer MLAG Link
      local_if_ip_address: 10.0.0.1/30
      local_if_disable_spanning_tree: true
      peer_address: 10.0.0.2
      peer_link_if: Port-Channel10
      peer_link_mode: trunk
      peer_link_lacp_mode: active
      peer_link_enable: true
      peer_link_members:
        - Ethernet3
        - Ethernet4

Sample host_vars/leaf2.example.com

    host: "{{ inventory_hostname }}"
    username: admin
    password: admin
    use_ssl: no
    authorize: yes
    transport: cli
    
    no_log: true
    
    mlag:
      mlag_domain_id: mlag1
      mlag_trunk_group: mlagpeer
      mlag_shutdown: false
      local_if_vlan: Vlan1024
      local_if_ip_address: 10.0.0.2/30
      local_if_disable_spanning_tree: true
      peer_address: 10.0.0.1
      peer_link_if: Port-Channel10
      peer_link_mode: trunk
      peer_link_lacp_mode: active
      peer_link_enable: true
      peer_link_members:
        - Ethernet3
        - Ethernet4

A simple playbook to enable MLAG on your leafs, leaf.yml

    - hosts: leafs
      roles:
         - arista.eos-mlag

Then run with:

    ansible-playbook -i hosts leaf.yml
    ​    


Developer Information
------------

Development contributions are welcome. Please see *Arista Roles for Ansible - Development Guidelines* ([test/arista-ansible-role-test/README](test/arista-ansible-role-test/README.md)) for additional information, including how to develop and run test cases for role development.




License
-------

Copyright (c) 2015, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com
