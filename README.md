## Overview

Configuration for a Cisco 2960-CX Switch.

The aim is to be able to use Ansible to configure the switch and to 
explore NETCONF and other automatic configuration options.


## Connections

```
192.168.32.0/24            ________________
----------------port 12---|                |
                          | Cisco 2960-CX  |---- Serial Console
                          |________________|
```
						  
						  
## Initial setup

The switch was configured with hostname `cisco2960a` and an enable secret.
The Vlan1 interface has been setup with the following config

```
enable
configure terminal
ip default-gateway 192.168.32.4
interface Vlan1
ip address 192.168.32.7 255.255.255.0
```

## Configuring SSH
 
### Preparatory Steps

Execute the following commands at the IOS prompt

```
enable
configure terminal

hostname cisco2960a
ip domain-name my.domain.home
crypto key generate rsa
end
copy running-config startup-config
```

The following output will be generated

```
The name for the keys will be: cisco2960a.my.domain.home
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...
[OK] (elapsed time was 1 seconds)
```


### Enable SSH

Run the following commands 

```
enable
configure terminal
ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3
line vty 1 10
transport input ssh
end
```

### AAA

```
enable
configure terminal
aaa new-model
aaa authentication login default local
aaa authorization exec default local
aaa authorization network default local
username admin privilege 15 password 0 SomePassword
end
show running-config
copy running-config startup-config
```


## Tools

### Ansible

This is easiest to use via a Python virtual environment.

NOTE: Given that Python2 is EOL on 1st January 2020 all examples are using Python3

```
virtualenv matrix
. ./matrix/bin/activate
pip install ansible
```


## Ansible Scripts

### Setup

The `switch2960cx.yml` file under `group_vars` specifies connection details for 
the group which is defined in the `inventory/hosts` file.  This includes a connection password

The following shows how to encrypt a password with Vault so that it can be included in the group_vars file.

```
ansible-vault encrypt_string
New Vault password: 
Confirm New Vault password: 
Reading plaintext input from stdin. (ctrl-d to end input)
EnterSomePassword!vault |
          $ANSIBLE_VAULT;1.1;AES256
          31353031386139656534663633386430666662313863636235363965306663386362663639366130
          3165323937303436663334323337623733353562653137640a373030313031336162346663626237
          30653164626461326236393330373038386636306436653736313033656466363038313437333236
          6166306265663337630a333765363739303663396265643032333133613034623861323335396661
          3064
Encryption successful
```

Don't hit enter at the end of the password otherwise that LF (or CRLF) characters will get encrypted as well.  When I tried this, on a Mac, I had to press CTRL-D twice after entering the password.

The text block can now be copied verbatim into the group_vars file.


### Reading Interface Data

This is a quick test script to check that connectivity and configuration are correct.

Run using the following command

```
ansible-playbook --ask-vault-pass -i inventory/hosts read_interfaces.yml 
```

This simply prints out the lines of text from stdout from running an IOS command.



## References

### Ansible Documentation

* [Platform options](https://docs.ansible.com/ansible/latest/network/user_guide/platform_ios.html)
* [Creating single variable vault values](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html#single-encrypted-variable)
* [Working with inventories](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
* [Ansible ios_command module](https://docs.ansible.com/ansible/latest/modules/ios_command_module.html)
* [Debug module](https://docs.ansible.com/ansible/latest/modules/debug_module.html)
* [ios_facts module](https://docs.ansible.com/ansible/latest/modules/ios_facts_module.html)
* [Ansible loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
* [Ansible dict key/value pairs](https://docs.ansible.com/ansible/latest/plugins/lookup/dict.html)
* 

### Cisco Documentation

* [Configure users](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960cx_3650cx/software/release/15-2_4_e/configurationguide/b_1524e_consolidated_3560cx_2960cx_cg/b_1524e_consolidated_3560cx_2960cx_cg_chapter_0110110.pdf)

* [Configuring SSH](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960cx_3650cx/software/release/15-2_4_e/configurationguide/b_1524e_consolidated_3560cx_2960cx_cg/b_1524e_consolidated_3560cx_2960cx_cg_chapter_0110111.pdf)

* [Troubleshooting](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960cx_3650cx/software/release/15-2_3_e/configuration/guide/b_1523e_consolidated_2960cx_3560cx_cg/b_consolidated_152ex_2960-X_cg_chapter_0111011.pdf)

* [Configuring VLANs](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960x/software/15-0_2_EX/vlan/configuration_guide/b_vlan_152ex_2960-x_cg/b_vlan_152ex_2960-x_cg_chapter_011.html)
