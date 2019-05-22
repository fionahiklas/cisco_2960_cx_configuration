## Overview

Configuration for a Cisco 2960-CX Switch.

The aim is to be able to use Ansible to 

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
line vtp 1 10
transport input ssh
end
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



## References

### Cisco Documentation

* [Configure users](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960cx_3650cx/software/release/15-2_4_e/configurationguide/b_1524e_consolidated_3560cx_2960cx_cg/b_1524e_consolidated_3560cx_2960cx_cg_chapter_0110110.pdf)

* [Configuring SSH](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960cx_3650cx/software/release/15-2_4_e/configurationguide/b_1524e_consolidated_3560cx_2960cx_cg/b_1524e_consolidated_3560cx_2960cx_cg_chapter_0110111.pdf)


